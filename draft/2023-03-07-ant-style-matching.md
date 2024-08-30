---
title: Ant Style Pattern Matching
date: 2023-03-07 10:02:00 +0900
categories: [SlipBox, Spring]
tags: [PatternMatching, AntStyle]
publish: false
---

Ant-style matching is a pattern matching technique used in software development to match strings based on certain patterns. It gets its name from the use of wildcard characters that resemble the behavior of ants crawling along a path.

The mapping matches URLs using the following rules:

- `?` matches one character
- `*` matches zero or more characters
- `**` matches zero or more directories in a path
- {spring:[a-z]+} matches the regexp [a-z]+ as a path variable named "spring"

# References
- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/util/AntPathMatcher.html
- https://lng1982.tistory.com/169