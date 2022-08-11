---
layout: post
title:  "The C10K problem"
date:   2022-08-12 18:00:00 +0900
categories: tech backend
---
### The C10K problem
21세기 초반에 엔지니어들은 확장성 병목 현상에 직면합니다. 웹 서버가 10,000개 이상의 연결을 처리할 수 없었던 것이죠. 또한 Apache 서버의 경우 성능은 동시 연결 수에 반비례했습니다.

아파치는 pre-forked 또는 multi-process mode 두 가지 주요 모드로 실행되도록 구성할 수 있습니다. 어떤 방식이든 요청이 들어오면 하나의 쓰레드가 점유되고 리소스를 위해 경쟁합니다. 쓰레드가 증가하면 컨텍스트 전환도 증가합니다. 처리량이 높더라도 요청의 기간이 짧으면 동시 연결 제한에 도달하지 못할 수 있습니다. 그러나 각 요청의 기간이 10초로 변경되면 1,000TPS에서 10K 연결이 열려 시스템 성능이 저하됩니다.

어떤 현상이 있었는지 더 자세히 살펴보겠습니다. 새로운 패킷은 어떤 스레드가 패킷을 처리하는지 알아내기 위해 커널의 모든 10K 프로세스를 순회해야 합니다. select/poll에서는 이벤트가 있는 파일 디스크립터를 찾기 위해 선형 검색을 했기 때문입니다.

### 새로운 모델의 탄생
이러한 한계를 해결하기 위해 이벤트 모델이라고 하는 완전히 다른 프로그래밍 모델을 기반으로 하는 웹 서버가 등장했습니다. 대표적으로 nginx입니다.

새로운 모델에서는 요청 당 하나의 쓰레드를 생성하지 않습니다 코어 개수에 맞게 프로세스를 띄웁니다. 각 프로세스는 async IO를 통해 많은 수의 동시 연결을 처리합니다. 특히 각 프로세스는 파일 디스크립터 셋을 갖고 있습니다.  각 요청에 대해 하나씩, 효율적인 epoll 시스템 호출을 사용하여 이벤트를 처리합니다.

![image](https://user-images.githubusercontent.com/3898834/184175989-d5aea0ca-3553-42c0-8214-16fde63d0d25.png)

### 함께 읽어볼만한 글
- http://www.kegel.com/c10k.html
- [The Architecture of Open Source Applications (Volume 2): nginx](http://www.aosabook.org/en/nginx.html#fig.nginx.arch)
