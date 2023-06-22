---
title: "What is CDATA"
date: 2023-06-22 22:00:00 +0900
categories: [Level, Senior]
tags: [xml, CDATA]
---

- XML(eXtensible Markup Language)의 한 섹션.
- 내용물이 마크업이 아니라 텍스트로 다루어진다.
  - 마크업은 데이터의 구조와 formatting을 결정한다.
  - 그런데 마크업으로 해석하지 말고 그대로 문자나 데이터를 포함(escape)해야할 상황이 존재.
- "<", ">", "&"
- 다음 구분자로 감싸서 사용한다.

```
<![CDATA[ your character data here ]]>
```

- 예를 들어 xml 문서에서 특별한 문자를 가지고 있는 description을 추가하는 경우:

```
<description><![CDATA[This is an example with <angle brackets>, &amp; and "quotation marks".]]></description>
```

- CDATA 섹션은 평문자 데이터로 다루어진다.
- 그렇지 않으면 XML parser가 markup으로 해석.
- data를 있는 그대로 표현가능
