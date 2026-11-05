---
title: "What is Claim in JWT?"
date: 2023-06-26 22:00:00 +0900
categories: [Level, Junior]
tags: [Spring, Security]
publish: false
---

In the context of JSON Web Tokens (JWT), a claim refers to a piece of information included in the token payload. The payload contains statements about the entity (typically a user) and additional data related to the token.

Claims are represented as key-value pairs and provide information such as the identity of the user (subject), the expiration time of the token, and various other metadata. There are three types of claims: registered claims, public claims, and private claims.

1. Registered Claims: These are a set of predefined claims with standardized meanings, defined by the JWT specification. Some common registered claims include:

"iss" (Issuer): Identifies the entity that issued the token.
"sub" (Subject): Represents the subject or identity of the token.
"exp" (Expiration Time): Indicates the expiration time of the token.
"iat" (Issued At): Specifies the time when the token was issued.
"nbf" (Not Before): Defines the earliest time the token can be accepted.

2. Public Claims: These claims are not standardized but are defined by the application or organization using JWTs. Public claims are meant to be used for interoperability, so it's recommended to define them in a way that avoids clashes with other claim names.

3. Private Claims: These claims are custom claims specific to the application or organization and are meant for private use. They are not meant to be shared or used outside the context of that application.

Claims are encoded within the JWT payload as JSON objects. They provide relevant information that can be used to validate and process the token on the server side. However, it's important to note that the server should always verify the integrity of the token and perform additional checks before fully trusting the information provided in the claims.