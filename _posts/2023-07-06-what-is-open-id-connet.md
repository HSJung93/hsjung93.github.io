---
title: "OpenID Connect(OICD)"
date: 2023-07-06 22:00:00 +0900
categories: [Level, Junior]
tags: [Spring, Security]
---

OpenID Connect (OIDC) is an identity layer built on top of the OAuth 2.0 protocol. It provides a standardized way for clients (such as web applications or mobile apps) to authenticate users and obtain their identity information from an OpenID Connect Provider (OP). OIDC enables single sign-on (SSO) and allows users to authenticate using their existing accounts with identity providers such as Google, Facebook, or Microsoft.

In OIDC, the communication flow involves three main entities: the client application, the OpenID Connect Provider (OP), and the user. Here is a simplified overview of the flow:

1. User initiates the authentication process by accessing the client application.
2. The client application redirects the user to the OP's authentication endpoint, including the necessary parameters.
3. The OP authenticates the user through various mechanisms, such as username/password, multi-factor authentication, or social login.
4. Once authenticated, the OP generates an ID Token, which contains information about the user, and issues an access token (optional) to the client application.
5. The client application can validate the ID Token to ensure its authenticity and retrieve user information from it. The access token, if issued, can be used to make authorized API requests on behalf of the user.
6. The client application can store the user's authentication state and establish a session to provide SSO capabilities for subsequent requests.

OIDC provides a standardized set of claims (user information) that can be included in the ID Token, such as user ID, name, email address, and other profile information. It also supports optional scopes for requesting additional permissions or information from the user during authentication.

By leveraging OIDC, developers can offload the responsibility of user authentication to trusted identity providers, improving security and reducing the need for managing user credentials.