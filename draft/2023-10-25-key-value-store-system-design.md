---
title: "키-값 저장소 설계"
date: 2023-10-25 13:05:00 +0900
categories: [Level, Senior]
tags: [시스템 설계, DB]
publish: false
---

키-값 저장소란?
- 비 관계형 데이터베이스
- 고유 식별자(identifier)
- 키-값 쌍(key-value pair)

CAP Theorem
- Consistency: 어떤 서버라도 같은 데이터
- Availability: 장애 나더라도 응답 받음
- Partition Tolerance: 두 노드 간 통신 장애 있어도 시스템 동작
    - Partition: 두 노드 간의 통신 장애

P is given
- CP: 쓰기 연산 중단하고 응답 X
- AP: 정합하지 않은 응답

데이터 일관성 보장
- 정족수 합의 프로토콜
    - N: 사본 개수
    - W: 쓰기 연산 성공 응답 하한
    - R: 읽기 연산 성공 응답 하한
- 벡터 시계
    - D([Si, vi])
        - [Si, vi]가 있으면 vi를 증가시킨다.
        - 그렇지 않으면 새 항목 [Si, 1]를 만든다.
    - 단점
        - 클라이언트 구현이 복잡해진다.
        - [서버: 버전]의 순서쌍 개수가 급속히 증가한다.

장애 감지
- 가십 프로토콜(Gossip protocol)

장애 처리
- 일시적: 단서 후 임시 위탁(hinted handoff)
- 영구: 머클 트리
