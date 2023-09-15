---
title: "what is CDATA"
date: 2023-06-22 22:00:00 +0900
categories: [Level, Senior]
tags: [xml, CDATA]
---

CDATA stands for Character Data, and it is a section of an XML (eXtensible Markup Language) document where the content is treated as plain text rather than markup. In XML, markup is used to define the structure and formatting of the data, but there are situations where you need to include characters or data that would otherwise be interpreted as markup.

CDATA sections are used to escape such characters and indicate that the enclosed data should be treated as character data rather than markup. CDATA sections are typically used when you need to include special characters like angle brackets ("<" and ">"), ampersands ("&"), or quotation marks within an XML document.

A CDATA section is defined by wrapping the desired content within the following delimiters:

```
<![CDATA[ your character data here ]]>
```

For example, suppose you have an XML document and you want to include a description with special characters:

```
<description><![CDATA[This is an example with <angle brackets>, &amp; and "quotation marks".]]></description>
```

In the above example, the text within the CDATA section will be treated as plain character data, and the special characters within it will not be interpreted as markup by the XML parser.

CDATA sections are useful when you want to include content that may contain reserved XML characters without the need for escaping or encoding them. It allows you to represent the data as-is without worrying about the XML syntax rules.
