---
title: "b3-propagation"
date: 2023-09-15 00:00:00 +0900
categories: [Level, Junior]
tags: [Zipkins, b3-propagation]
publish: false
---

# b3-propagation
- B3 Propagation is a specification for the header "b3" and those that start with "x-b3-"
- used for trace context propagation across service boundaries
- propagated in-process, and eventually downstream (often via http headers)
- should be collected and reported to the tracing system (usually Zipkin) or not.

# Identifiers
- Trace identifiers are 64 or 128-bit, but all span identifiers within a trace are 64-bit. 
- TraceId: The TraceId is 64 or 128-bit in length and indicates the overall ID of the trace. Every span in a trace shares this ID.
- SpanId: The SpanId is 64-bit in length and indicates the position of the current operation in the trace tree. The value should not be interpreted: it may or may not be derived from the value of the TraceId.
- ParentSpanId: The ParentSpanId is 64-bit in length and indicates the position of the parent operation in the trace tree. When the span is the root of the trace tree, there is no ParentSpanId.

# Http Encodings
- There are two encodings of B3
    - Multiple header encoding uses an `X-B3-` prefixed header per item in the trace context.
    - Single header delimits the context into into a single entry named `b3`.

## Multiple Headers
- B3 attributes are most commonly propagated as multiple http headers. All B3 headers follows the convention of `X-B3-${name}` with special-casing for flags.
- The `X-B3-TraceId` header is encoded as 32 or 16 lower-hex characters. 
- The `X-B3-SpanId` header is encoded as 16 lower-hex characters. 
- The `X-B3-ParentSpanId` header may be present on a child span and must be absent on the root span. 

## Single Header
- A single header named `b3` standardized in late 2018 for use in JMS and w3c `tracestate`. 
- In simplest terms `b3` maps propagation fields into a hyphen delimited string.
- `b3={TraceId}-{SpanId}-{SamplingState}-{ParentSpanId}`, where the last two fields are optional.
For example, the following state encoded in multiple headers:
```
X-B3-TraceId: 80f198ee56343ba864fe8b2a57d3eff7
X-B3-ParentSpanId: 05e3ac9a4f6e3b90
X-B3-SpanId: e457b5a2e4d86bd1
X-B3-Sampled: 1
```
Becomes one `b3` header, for example:
```
b3: 80f198ee56343ba864fe8b2a57d3eff7-e457b5a2e4d86bd1-1-05e3ac9a4f6e3b90
```