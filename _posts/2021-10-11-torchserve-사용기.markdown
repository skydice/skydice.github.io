---
layout: post
title:  "torchserve 사용기"
date:   2021-10-11 18:00:00 +0900
categories: tech
---
## 개요
모델을 서빙할 때마다 Flask로 했었는데 반복되다 보니 여간 귀찮은 일이 아니었다. 단순히 서빙하는 것 이외에 에러 핸들링, 서빙, 모니터링 등 할 것이 많기 때문이다. 이미 다른 기술 팀에서는 라이브러리를 사용해 운영의 효율을 높이고 있는 것처럼 보였다. 나도 가장 범용적으로 사용되고 있는 것으로 보이는 torchserve를 적용해봤다.
이 글의 목적은 torchserve에 대한 overview이자 사용기이다. 해당 기술에 대한 기초적인 이해에 초점을 맞췄다.
## torchserve 장점
* Batch Inference 구현 (여러 요청을 모아 batch 방식으로 처리하는 방법)
* BaseHandler를 상속받은 custom handler을 구현하는 것만으로 손쉽게 모델 서빙
* 로깅, 메트릭, 벤치마크 지원
* 쿠버네티스 지원 (KFserving)
## torchserve 구조
* 프론트엔드는 자바로 백엔드는 파이썬으로 되어 있음
* 프론트엔드는 요청과 응답을 처리하며 요청을 Batch 형태로 만들어 백엔드로 전달
* 백엔드에는 모델 워커가 떠서 요청을 처리 (여러 다른 모델이 뜰 수 있음)
* 기본적으로 두 개 포트가 뜨는데 8080이 추론을 위한 API를 호출할 때 사용하고 8081은 매니지먼트를 위한 API를 호출할 때 사용
## mar 파일 말기
PyTorch에서 모델은 mar 파일로 아카이브 된다. 모델에 관련된 파일들이 아카이브 된 형태이다. 이 파일을 단위로 모델이 서빙되며 구성 요소는 아래와 같다.
* 모델 (.py)
* state_dict (.pth)
* 결과 맵핑 (index_to_name.json) - 결과로 나온 모델의 index를 텍스트로 맵핑하기 위해 필요
* 핸들러 (BaseHandler 구현) 
### 샘플
* example로 제공되는 densenet161의 mar는 아래와 같은 방법으로 말 수 있음 [참조](https://github.com/pytorch/serve/tree/master/examples/image_classifier/densenet_161)
```
torch-model-archiver --model-name densenet161 --version 1.0 --model-file ./examples/image_classifier/densenet_161/model.py --serialized-file ./examples/image_classifier/densenet161-8d451a50.pth --export-path ./model-store --extra-files ./model-server/examples/image_classifier/index_to_name.json --handler image_classifier
```
### 핸들러
* 기본적으로 제공하는 핸들러 이외에 사용자 정의에 따라 커스텀 핸들러를 만들 수 있음
* 커스텀 핸들러가 `BaseHandler`를 상속하지 않으면 반드시 `initialize`와 `handle` 메소드를 구현해야 함
* 일반적으로 `BaseHandler`를 상속하는데 이 경우 `initialize`, `preprocess`, `inference`, `postprocess` 등의 메소드 등을 구현
* `handle`에서 정의한 메소드들을 순차적으로 실행시키며 동작
### 예제 Handler 살펴보기
* 아래는 github에서 공식적으로 제공하고 있는 example이 어떻게 구현되어 있는지를 간략하게 살펴본다.
#### Vision Handler의 Preprocess
```
def preprocess(self, data):
  ...
    for row in data:
        image = row.get("data") or row.get("body")
        if isinstance(image, str):
            # if the image is a string of bytesarray.
            image = base64.b64decode(image)

        # If the image is sent as bytesarray
        if isinstance(image, (bytearray, bytes)):
            image = Image.open(io.BytesIO(image))
            image = self.image_processing(image)
        else:
            # if the image is a list
            image = torch.FloatTensor(image)

        images.append(image)

    return torch.stack(images).to(self.device)
```
* json 형태의 데이터의 경우 byte array를 base64로 인코딩 해야 하기 때문에 만약 `str` 형태로 데이터가 들어오면 base64 디코딩
* 이미지가 여러 개 들어오면 batch inference를 위해 `torch.stack()`을 통해 Tensor로 변환
#### Base Classifier의 Initialize
```
def initialize(self, context):
  # 모델 파일 정의
  properties = ctx.system_properties
  model_dir = properties.get("model_dir")
  self.model = torch.load(os.path.join(model_dir, 'model_file.pth'))
  ...
  mapping_file_path = os.path.join(model_dir, "index_to_name.json")
  self.mapping = load_label_mapping(mapping_file_path)
```
* `index_to_name.json`을 읽어 `self.mapping`에 할당
* `load_label_mapping`는 torchserve에서 제공
#### Text Classifier의 Postprocess
```
def postprocess(self, data):
    ...
    data = F.softmax(data)
    data = data.tolist()
    return map_class_to_label(data, self.mapping)
```
* `initialize`에서 정한 `self.mapping`을 사용하여 index를 라벨로 바꿈
* `map_class_to_label`는 torchserve에서 제공
## 배치 처리하기
* GPU 활용도를 올리기 위해 배치 인퍼런스를 기본적으로 제공
* `batch_size`는 한번에 처리할 최대 쿼리 개수를 의미
* `max_batch_delay`는 `batch_size`개의 요청이 들어오는 것을 기다릴 최대 시간을 의미 
* 즉 `max_batch_delay`가 지나면 모인 쿼리를 모두 처리 
* 이전에 `batch_size`만큼 개수가 차면 기다리지 않고 처리
* 이는 management API로 설정 가능 (그러나 등록 시에 설정하는 것으로 이미 등록된 모델의 설정을 바꿀 수는 없음)
```
curl -X POST "localhost:8081/models?url=resnet-152.mar&batch_size=8&max_batch_delay=50"
```
* 이와 다른 방법으로는 `config.properties`에 다음과 같이 설정 (`config.properties`는 torchserve를 실행할 때 필요한 설정 파일)
```
models={\
  "resnet-152": {\
    "1.0": {\
        "defaultVersion": true,\
        "marName": "resnet-152.mar",\
        "minWorkers": 1,\
        "maxWorkers": 1,\
        "batchSize": 8,\
        "maxBatchDelay": 50,\
        "responseTimeout": 120\
    }\
  }\
}
```
## KFServing로 서빙하기
* KFServing의 HTTP/REST API는 기본적으로 JSON을 채택하고 있음
* 따라서 해당 프로토콜을 사용하기 위해서는 base64 인코딩을 사용해야 함 [참조](https://github.com/kserve/kserve/blob/master/docs/predict-api/v2/required_api.md)
* Raw 데이터로 추론을 원하는 경우 커스텀 이미지를 사용해야 함  [참조](https://github.com/kserve/kserve/tree/37af39054499caf9145664a48981740ca4ce14f5/docs/samples/v1beta1/custom/torchserve)
* KFServing을 위해선 다음의 두 가지 과정이 필요
1. Persistent Volume Container를 만듦 [참조](https://github.com/kserve/kserve/blob/master/docs/samples/v1beta1/torchserve/model-archiver/README.md) - 아래 형태로 구성되어 있어야 함
```
├── config
│   ├── config.properties
├── model-store
│   ├── densenet_161.mar
│   ├── mnist.mar
```
2. InferenceService를 생성하여 서빙 - 1에서 띄운 PVC의 경로를 아래와 같이 명시
```
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: "torchserve"
spec:
  predictor:
    pytorch:
      storageUri: gs://kfserving-examples/models/torchserve/image_classifier
```
## 사용 후기
사실상 핸들러만 구현하면 모델을 쉽게 서빙할 수 있었다. 그러나 충족할 수 없는 요구 사항이 들어올 수도 있다는 생각이 들었다. 예를 들면 음성 처리 시 VAD로 음성 구간을 검출한 뒤 추론한다던가 하는 복잡한 요구 사항 말이다. 성능 테스트까지 안해봤지만 쥐어짜내야 하는 경우 구현에 답답함을 느낄 수 있을 것 같기도 했다. 일단 지금 하고 있는 프로젝트는 투입해보고 후기를 다시 남기겠다.
