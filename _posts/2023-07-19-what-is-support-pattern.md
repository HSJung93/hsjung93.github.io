---
title: "Support Pattern"
date: 2023-07-19 19:00:00 +0900
categories: [Level, Senior]
tags: [Java, DesignPattern]
---

The Support design pattern, also known as the Chain of Responsibility pattern, is a behavioral design pattern that allows an object to pass a request along a chain of potential handlers until one of them handles the request. The main idea is to decouple the sender of a request from its receivers and give multiple objects a chance to handle the request.

Here's an example of the Support design pattern implemented in Java:

```java
public abstract class SupportHandler {
    private SupportHandler nextHandler;

    public void setNextHandler(SupportHandler nextHandler) {
        this.nextHandler = nextHandler;
    }

    public void handleRequest(Request request) {
        if (canHandle(request)) {
            processRequest(request);
        } else if (nextHandler != null) {
            nextHandler.handleRequest(request);
        } else {
            System.out.println("Unable to handle the request.");
        }
    }

    protected abstract boolean canHandle(Request request);

    protected abstract void processRequest(Request request);
}

```

In this example, SupportHandler is an abstract class representing a support handler. Each concrete support handler subclass will implement the canHandle() and processRequest() methods according to their specific responsibilities.

```java
public class TechnicalSupportHandler extends SupportHandler {
    @Override
    protected boolean canHandle(Request request) {
        return request.getType() == RequestType.TECHNICAL;
    }

    @Override
    protected void processRequest(Request request) {
        System.out.println("Technical support handling request: " + request.getDescription());
    }
}

```
The TechnicalSupportHandler is a concrete subclass of SupportHandler that handles technical support requests.

```java
public class BillingSupportHandler extends SupportHandler {
    @Override
    protected boolean canHandle(Request request) {
        return request.getType() == RequestType.BILLING;
    }

    @Override
    protected void processRequest(Request request) {
        System.out.println("Billing support handling request: " + request.getDescription());
    }
}

```

The BillingSupportHandler is another concrete subclass of SupportHandler that handles billing support requests.

```java
public class Application {
    public static void main(String[] args) {
        // Creating support handlers
        SupportHandler technicalSupportHandler = new TechnicalSupportHandler();
        SupportHandler billingSupportHandler = new BillingSupportHandler();

        // Setting up the chain of responsibility
        technicalSupportHandler.setNextHandler(billingSupportHandler);

        // Sending requests
        Request technicalRequest = new Request(RequestType.TECHNICAL, "Help with software installation");
        Request billingRequest = new Request(RequestType.BILLING, "Payment processing issue");

        technicalSupportHandler.handleRequest(technicalRequest);
        technicalSupportHandler.handleRequest(billingRequest);
    }
}

```

In the Application class, we create instances of TechnicalSupportHandler and BillingSupportHandler. We then set up the chain of responsibility by setting billingSupportHandler as the next handler for technicalSupportHandler. Finally, we send requests to the technicalSupportHandler, which will pass the requests along the chain until they are handled by the appropriate handler.

This way, the responsibility for handling different types of requests is distributed among the support handlers, allowing flexibility and dynamic handling of requests at runtime.