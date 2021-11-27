---
layout: post
title:  "torchserve에서 Custom Handler 테스트 하기"
date:   2021-10-31 18:00:00 +0900
categories: tech
---
torchserve는 핸들러 파일만 구현하면 서빙할 수 있습니다. Custom Handler는 어떻게 테스트 할 수 있을까요? torchserve에서는 로컬 테스트를 위해 MockContext를 제공합니다. 이를 통해 Custom Handler를 충분히 테스트 해볼 수 있습니다.

## MockContext
* MockContext는 다음의 인자를 받아 생성됩니다.

```
def __init__(self,
             model_pt_file='model.pt',
             model_dir='ts/torch_handler/unit_tests/models/tmp',
             model_file='model.py',
             gpu_id='0',
             model_name="mnist"):
```

## 샘플
* 아래와 같이 handler를 생성할 수 있습니다. `handle` 등을 호출해 테스트합니다.

```
context = MockContext(
    model_pt_file=model_pt_file,
    model_dir=os.path.join(project_path, 'package_files'),
    model_name=model_name
)

mnist_handler = Handler()
mnist_handler.initialize(context)
```
