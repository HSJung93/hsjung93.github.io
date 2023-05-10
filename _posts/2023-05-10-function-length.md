---
title: Function Length
date: 2023-05-10 20:00:01 +0900
categories: [SlipBox, Code]
tags: [Function, Length]
---

- how long a function should be.
- when should we enclose code in its own function?
- based on length, such as functions should be no larger than fit on a screen
- based on reuse - any code used more than once should be put in its own function, but code only used once should be left inline.
- separation between intention and implementation.
    - If you have to spend effort into looking at a fragment of code to figure out what it's doing, then you should extract it into a function and name the function after that “what”.
    - That way when you read it again, the purpose of the function leaps right out at you, and most of the time you won't need to care about how the function fulfills its purpose - which is the body of the function.
- an example that Kent Beck showed me from the original Smalltalk system.
    - Smalltalk's graphics class had a method for this called 'highlight', whose implementation was just a call to the method 'reverse'
    - The name of the method was longer than its implementation - but that didn't matter because there was a big distance between the intention of the code and its implementation.
    - I remember people objecting to having an isEmpty method for a list when the common idiom is to use aList.length == 0. 
    - But here using the intention-revealing name on a function may also support better performance if it's faster to figure out if a collection is empty than to determine its length.
- Small functions like this only work if the names are good, so you need to pay good attention to naming.

# Reference
- https://www.oreilly.com/library/view/five-lines-of/9781617298318/
- https://medium.com/@kanani-nirav/the-five-lines-of-code-principle-why-less-is-more-in-programming-12ff4446205
- https://martinfowler.com/bliki/FunctionLength.html#footnote-highlight
- https://tangoblog.tistory.com/4