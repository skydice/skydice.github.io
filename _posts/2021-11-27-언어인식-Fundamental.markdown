---
layout: post
title:  "언어인식 Fundamental"
date:   2021-11-27 18:00:00 +0900
categories: tech deeplearning
---
# 언어인식 Fundamental
이 글은 2013년 공개된 Spoken Language Recognition: From Fundamentals to Practice의 앞 부분을 요약한 글입니다.

## 언어인식이란?
*언어인식*은 음성이 어떤 언어인지 찾아내는 것입니다. 음성 번역, 다언어 음성인식, 음성 문서 검색 등 넓은 범위의 어플리케이션에서 사용되고 있습니다. 
실제로 음성 언어인식은 기계가 오류 없이 음성을 텍스트로 옮겨 쓸 수 있다는 보장이 없기 때문에 문자 언어인식보다 훨씬 어렵습니다. 
우리는 인간이 청각 시스템에 내재된 지각적 또는 정신 음향적 과정을 통해 언어를 인식한다는 것을 알고 있습니다. 그러므로 청취자들이 사용하는 인식 단서는 언어인식에서 주요한 영감의 원천이 됩니다.

## 언어인식의 중요한 단서
언어인식에는 다음 두 가지 맥락이 있습니다.
* *prelexical 정보* 음소군, 음운론, 리듬, 억양
* *lexical semantic 지식* 어휘적 의미 지식

언어를 잘 모르는 유아들도 prelexical 정보로 언어를 구분합니다. 어른들은 친숙하지 않은 두 언어를 대할 때 prelexical 정보만 사용할 수 있습니다. 우리는 잘 모르는 이탈리아어와 아랍어를 듣고 두 언어를 쉽게 구분할 수 있습니다. 

유아가 언어를 잘 알게 되거나 성인이 익숙한 언어를 다루면 lexical semantic 지식이 중요해집니다. 언어인식에서 어떤 것이 단서가 되는지는 언제나 논쟁 거리였습니다. 여러 연구들에서 acoustic과 phonotactic이 가장 효과적인 언어 단서라고 주장하고 있습니다.

* *acoustic* 물리적 소리 패턴
* *phonotactic* 음소적 제약

## 언어의 특성화 원리
청취자는 익숙하지 않은 언어에 대해 알고 있는 언어를 참고하여 주관적으로 판단합니다. 이전에 노출이 거의 없는 언어라면  많은 사전 지식 없이도 언어를 효과적으로 식별할 수 있습니다. 이 경우 언어를 특징짓기 위해 두드러진 음성, 운율 특성에 의존합니다.

![B430DE17-5E15-4370-AD8F-67D486AADF58](https://user-images.githubusercontent.com/3898834/143675896-f4e5adec-a867-4e63-a98f-8e346cc23855.png)

추상화 수준에 따라 언어인식 단서는 일반적으로 acoustic phonetics, phonotatics, prosodic, words and syntax으로 나뉩니다.

### Acoustic Phonetics
구체적인 acoustic events로서의 음성을 phones라고 부르는 반면, 언어 체계에서 실체로서의 음성은 음소(phonemes)라고 합니다.
음성 단서의 사용은 언어가 부분적으로 중복되는 음소 집합을 가지고 있다는 가정에 기초합니다. 실제로 전 세계에 6,906개의 언어가 있지만 이 언어들의 모든 소리를 나타내기 위해 필요한 총 phone의 개수는 200개에서 300개 사이입니다. 언어에서 일반적인 phone들은 함께 그룹화되며 IPA에서 동일한 기호가 부여됩니다.
언어에서 사용되는 phonemes 수는 약 15개에서 50개 사이이며, 대다수는 각각 약 30개의 음소를 가지고 있습니다. 

* 영어: 24개의 자음과 14개의 모음
* 만다린어: 21개의 자음과 10개의 모음
* 스페인어: 18개의 자음과 5개의 모음

![82A15D36-B815-4BB0-81CE-7129DA9CF8CD](https://user-images.githubusercontent.com/3898834/143675906-7697cfa0-5012-4552-9347-78caa41766e4.png)

언어마다 음성 레퍼토리는 다르지만 언어는 일부 공통 음소를 공유할 수 있습니다. 음성 레퍼토리의 이러한 차이는 각 언어마다 고유한 phonemes, 즉 acoustic–phonetic feature distributions가 있음을 의미합니다.

### Phonotactics
각 언어는 다른 phonemes의 조합을 지배하는 고유한 lexical–phonological 규칙 집합을 가지고 있습니다. 언어 간에 phonemes이 공유될 수 있지만, 그 순차적 패턴에 대한 통계는 언어마다 다릅니다. 한 언어에서 자주 발생하는 일부 phone 시퀀스는 다른 언어에서는 드물 수 있습니다. 예를 들어 /fl/, /pr/ 및 /str/와 같은 자음 군집은 일반적으로 영어 단어에서 관찰되지만 중국어에서는 허용되지 않습니다. 이러한 음성적 제약은 phone n-gram 모델로 특징지어질 수 있습니다.

### Prosody
Prosody는 일반적으로 스트레스, 지속 시간, 리듬, 억양과 같은 말을 실행하는 데 있어 suprasegmental 특징을 나타냅니다. 상호 연관된 prosodic 특성 집합은 모두 언어의 중요한 특성입니다. 

* stress-timed languages: English, Germanic
* syllable-timed languages: French, Hindi, Korean
* mora-timed languages: Japanese

Prosody는 넓게 언어를 구분하는데 유용합니다. (e.g., tonal versus nontonal languages). 
phonotactics 특징이 prosodic 특징보다 언어인식에서 더 중요함으로 자세히 다루지 않습니다.

### Words and syntax 
* 음운 체계(phonological system): 단어나 형태소(morphemes)를 형성하기 위해 기호가 사용되는 방식
* 통사적 체계(syntactic system): 단어와 형태소가 어떻게 결합되어 구절과 말을 형성하는지

각 언어는 언어인식에 사용될 수 있는 고유한 음운 체계(phonological system)와 통사적 체계(syntactic system)를 가지고 있습니다. 이는 고유한 단어 목록 또는 단어 n-gram 집합의 특징을 나타냅니다. 그러므로 lexical 접근은 언어인식에 매우 적합한 접근으로 보입니다. 그리고 언어를 잘 알 때 사람이 듣기를 잘하는 점에서도 뒷받침 됩니다. 

## 결론
언어인식에서 중요한 건 acoustic–phonetic feature distributions과 그것들의 배열이 만들어 내는 특징입니다. phoneme과 그 배열을 중요한 단서로 삼는 음성인식 모델링 방법을 차용할 수 있습니다. 특히 시퀀셜 정보를 담는 RNN 모델이 성능이 좋을 것으로 예상합니다. 이에 대한 리서치는 차후 공유할 예정입니다.
