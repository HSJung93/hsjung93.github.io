---
title: 카프카의 isolation level
date: 2023-02-20 10:04:00 +0900
categories: [SlipBox, Spring]
tags: [Kafka, Isolation]
---

트랜잭션을 고려하여 쓰여진 메세지를 어떻게 읽을 것인가에 대한 옵션이다.
`read_committed`일 경우, `consumer.poll()`은 commit된 트랜잭션 메세지만 가져온다.
`read_uncommited`일 경우, `consumer.poll()`은 트랜잭션이 중단된 메세지들을 포함한 모든 메세지들을 가져온다.
즉 트랜잭션 메세지는 두 경우 모두 읽으며, 기본값은 `read_uncommited` 이다.