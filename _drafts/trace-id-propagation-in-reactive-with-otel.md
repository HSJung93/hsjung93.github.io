---
title: "Reactive 에서 trace-id 가 사라질 때: OpenTelemetry 와 Ansible"
date: 2026-12-31 08:00:00 +0900
categories: [개발, 백엔드]
tags: []
---

<!-- 목차 스켈레톤. 재료: ~/p-review/13_포스팅_목차.md B6 -->

## 증상: 요청 하나가 로그에서 여러 조각으로 끊긴다. 스레드 홉핑과 ThreadLocal 의 한계



## 원리: B3 헤더(멀티/싱글), context propagation 이 Reactor 에서 어떻게 이어지나 (초안 재활용)



## 선택: 코드에 수동 전파 vs OpenTelemetry Agent 자동 계측. Agent 를 택한 이유



## 배포: Ansible 로 Agent 와 설정을 모든 인스턴스에 일관되게. 플레이북 구성 요지



## 검증: 게이트웨이 → 서비스 → 외부 호출까지 하나의 trace 로 이어진 화면



## 교훈: 관측 가능성은 "나중에 붙이는 것"이 아니라 배포 파이프라인의 일부
