---
title: Collection Synchronization for WebDAV
date: 2022-10-31 20:00:00 +0900
categories: [SlipBox, Calendar]
tags: [WebDAV, CollectionSynchronization]
---
# WebDAV Synchronization
## Overview
- In order to synchronize the contents of a collection between a server and client, the server provides the client with a synchronization token each time the synchronization report is executed.
- That token represents the state of the data being synchronized at that point in time.
- The client can then present that same token back to the server at some later time, and the server will return only those items that are new, have changed, or were deleted since that token was generated.
- The server also returns a new token representing the new state at the time the report was run.