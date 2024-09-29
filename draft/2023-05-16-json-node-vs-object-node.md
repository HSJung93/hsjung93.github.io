---
title: JsonNode vs ObjectNode
date: 2023-05-16 22:00:00 +0900
categories: [메모상자, 자바]
tags: [라이브러리, 잭슨]
publish: false
---

Jackson 라이브러리는 자바에서 JSON 데이터를 다룰 때에 주로 사용되는 라이브러리이다. JsonNode와 ObjectNode 모두 이 Jackson 라이브러리가 제공하는 클래스이다. 그러나 JsonNode는 어떤 JSON 노드도 표현하는 제너릭 클래스이고, ObjectNode는 JSON 객체들 중 하나를 특정하는 클래스이고 객체 프로퍼티를 조작하는 메서드들을 포함한다. 

JsonNode는 Json 구조 하에서 한 노드를 나타내는 추상 클래스이다. object, array, string, number, boolean, null 등의 JSON 데이터 타입을 구현한다. 트리 구조의 JSON 데이터를 접근하고 조작하는 방법을 제공한다.

ObjectNode는 JsonNode의 서브 클래스이며 한 JSON 객체를 특정하여 구현한다. JSON 객체를 위한 추가적인 메서드를 가지고 있어 프로퍼티를 더하고 제거하고 수정할 수 있다. ObjectNode는 JSON 객체 안에서 키 밸류를 조작할 수 있게 해준다. 