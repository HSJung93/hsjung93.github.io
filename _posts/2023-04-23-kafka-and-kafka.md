---
title: 카프카와 카프카
date: 2023-04-23 15:29:00 +0900
categories: [에세이, 글 쓰는 개발자]
tags: [카프카]
---

> 어느 날 아침 그레고르 잠자는 불안한 꿈을 꾸다가 깨어나 보니 침대 속에서 흉측한 갑충으로 변해 있었다. 그는 철갑처럼 단단한 등을 바닥에 대고 누워 있었고 머리를 약간 쳐들자 활 모양의 각질로 칸칸이 나뉜 둥그런 갈색 배가 보였다. 몸뚱이에 비해 형편없이 가는 수많은 다리들이 속수무책으로 버둥거리며 그의 눈앞에서 어른거렸다. '이게 무슨 일이지?' 그는 생각했다. 꿈은 아니었다.
{: .prompt-info }

대학교 수업에서 읽을 소설을 고르고 있노라면, 항상 카프카는 매력적인 선택지였다. 유명한 평론가가 20세기를 대표하는 작가로 꼽았다는 사실이나, 40세의 짧은 인생으로 요절한 천재의 아우라 때문만은 아니었다. 그 시절 나에게 카프카가 매력적이었던 가장 큰 이유는 바로 소설의 두께에 있었다. 내노라하는 소설들을 읽어보고 싶지만 소설을 읽다가 노는 시간을 뺏기기는 싫은, 딱 그 정도의 얄팍한 각오을 가진 나에게 \<변신\>, \<심판\>, \<성\>, \<시골의사\> 처럼 짧고 굵은 카프카의 소설들은 참 잘 쓴 소설들이었던 것이다. 물론 이런 꾀부림들은 밑도 끝도 없이 사람이 벌레로 변한다던가 하는 첫문장을 읽고 나면 이거 잘못 걸렸다는 헛웃음으로 마무리되곤 했다.

그런데 개발자가 되고 보니 개발자들의 세계에도 "카프카"가 있었다. 개발자들의 세계에서 카프카는 대규모의 데이터 스트림을 안정적이고 빠르게 분산 처리할 수 있도록 개발된 오픈소스 솔루션이다. 앞선 작가로서의 카프카와 구분하기 위함인지는 모르겠지만, 이 카프카를 지칭할 때에는 오픈 소스 소프트웨어 그룹인 아파치 소프트웨어 재단의 이름을 붙여, "아파치 카프카"라 부르기도 한다. 그런데 사실은 두 카프카와 카프카를 굳이 구분해서 부를 필요가 없을지도 모른다. 왜냐하면 사실 아파치 카프카라는 이름 자체가 작가 카프카에서 따온 것이기 때문이다.

![](/assets/img/kafka-writer.jpg)

> 프란츠 카프카(Franz Kafka, 1885년 12월 31일 ~ 1924년 12월 31일)
{: .prompt-tip }

카프카의 이름이 카프카인 이 놀라운 우연의 원인은 생각보다 간단하다. 카프카는 2011년 초 링크드인의 Jay Kreps라는 개발자에 의해 개발되어 오픈소스로 공개되었다. 글로벌 버전의 네이버 지식인 쯤 되는 Quora 라는 사이트에서 그의 글을 찾아보면, 그가 카프카를 카프카라고 부르게된 이유를 찾아볼 수 있다. Jay Kreps는 대학교 시절 많은 문학 수업을 들었는데, 작가들 중에서도 프란츠 카프카라는 이름이 듣기 좋았으며, 자신이 개발한 솔루션이 쓰기 작업에 최적화된 시스템(a system optimized for writing)이기 때문에 카프카의 이름을 따오게 되었다고 밝히고 있다. 모르긴 몰라도 아파치 카프카는 "잘 쓰기" 위하여 개발된 시스템인 것이다. 

개발자의 입장에서 "쓰기"란 무엇인가? 작가가 정보를 남기기 위하여 종이에 글을 쓰듯(혹은 워드 프로세서에 타이핑을 하듯), 컴퓨터 또한 정보를 남기기 위하여 "쓰기" 작업을 한다. 작가가 남긴 글은 다른 사람들이 읽을/구독할 수 있도록 출판된다. 컴퓨터 또한 마찬가지이다. 컴퓨터가 남긴 정보는 다른 컴퓨터가 읽을(subscribe) 수 있도록 출판(publish)된다. 이 둘은 단순히 비유로서만 유사한 것이 아니다. 실제로 개발자들 사이에서도 구독과 출판이라는 출판 업계에서 쓸 법한 표현들을 사용한다. 그리고 이 컴퓨터들 간의 읽기와 쓰기 작업을 굉장히 안정적으로 빠르게 처리해주는 솔루션이 바로 카프카이다. 

![](/assets/img/apache-kafka.png)

> 아파치 카프카의 구조
{: .prompt-tip }

카프카에는 이 쓰기 작업을 안정적이고 빠르게 처리하기 위한 수많은 기술적인 고려사항들이 집약되어 있다. 쓰기 작업을 수행하는 발행자(Publisher)와 구독자(Subscriber)의 Pub-Sub 구조, 발행자와 구독자 사이에서 효율적으로 정보를 전달하는 브로커, 브로커들 사이에서 정보를 복제하고 병렬적으로 관리하는 토픽 등이 그것이다. RabbitMQ나 ActiveMQ 등 이전까지 메세지를 운반하는 수많은 큐 구조의 솔루션들이 난립했었지만, 카프카는 유려하고 정확하게 시장의 니즈를 파악해 다음 경지를 보여주었다. 카프카로 하여금 전임자를 대체할 수 있도록 한 이 기술적인 아름다움을 보다 적어보고 싶지만, 이 글에서 상정하는 독자는 개발자가 아니므로 카프카의 구조와 핵심적인 용어들을 보여주는 수준에서 멈추고자 한다.

결국 카프카는 어디까지나 정보를 효율적으로 "쓰기"위한 통로인 셈이다. 20세기 프란츠 카프카가 문학계에 미친 영향 만큼이나, 2023년 현재 아파치 카프카 또한 오픈 소스 생태계에 지대한 영향을 미치고 있는 것으로 보인다. 많은 기업들에서 이미 상용 서비스에 카프카를 도입하여 운영하고 있고, 따라서 기존 기술 스택에 카프카를 추가하는 방법이나 운영 상의 노하우 등은 구글이나 유튜브를 검색해보면 어렵지 않게 살펴볼 수 있다. 아무리 하루가 다르게 새것이 옛것이 되어버리는 IT 업계라지만, 카프카의 질주는 당분간 계속 될것만 같다. 눈부신 작가로서의 성과가 무색하게도 짧은 일생으로 끝맺은 작가로서의 카프카와는 다르게. 
 
# References
- 카프카 영문 위키피디아: https://en.wikipedia.org/wiki/Apache_Kafka
- 카프카와 카프카의 관계에 대한 Jay Kreps의 답변: https://www.quora.com/What-is-the-relation-between-Kafka-the-writer-and-Apache-Kafka-the-distributed-messaging-system/answer/Jay-Kreps