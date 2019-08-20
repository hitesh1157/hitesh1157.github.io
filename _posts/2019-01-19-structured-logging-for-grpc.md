---
layout: post
title: Structured Logging for gRPC requests
published: true
---

When you have a request-response gateway, you might want to log those incoming requests' parameters. By parameters, I mean client headers which for example include client's IP, request timestamp in local format etc. Maybe you want to add an identifier to that request on the server so that if that request gets passed around, you still be able for search for that identifier and see the lifecycle of that request, or maybe you want to use a global error handler function etc.

I describe how I implemented logging for my gRPC server running on GKE. I had a java server running on GKE.

First and most important thing, use `java.util.logging.Logger` and not `System.out.*`. Because in `println(String)` you will have to add timestamp and logging level manually. You can read more about `Logger` [here](https://www.journaldev.com/977/logger-in-java-logging-example). So the idea of implementing this is to intercept every request and before you pass that request to appropriate RPC, you take that request extract all the identifying information and log it. The easiest way to start is to implement `ServerIntercepter` from `io.grpc` package.

```java
public class MyLoggingInterceptor implements ServerInterceptor {

    private Logger logger = Logger.getLogger("MyLoggingInterceptor");
    private static final Metadata.Key<String> REQUEST_ID_HEADER_KEY =
                Metadata.Key.of(LoggingConstants.REQUEST_ID, Metadata.ASCII_STRING_MARSHALLER);

    public LoggingInterceptor() {
        logger.setLevel(Level.INFO);
    }

    @Override
    public <ReqT, RespT> ServerCall.Listener<ReqT> interceptCall(ServerCall<ReqT, RespT> call, Metadata headers, ServerCallHandler<ReqT, RespT> next) {

        // Using timestamp as unique id for each RPC call
        Long receivedTime = new Date().getTime();
        headers.put(REQUEST_ID_HEADER_KEY, receivedTime.toString());
        logger.log(Level.INFO, String.format("%d: Call (MethodName=%s)", receivedTime, call.getMethodDescriptor().getFullMethodName()));
        return new ForwardingServerCallListener.SimpleForwardingServerCallListener<ReqT>(next.startCall(call, headers)) {

            Long time = 0L;

            @Override
            public void onComplete() {
                logger.log(Level.INFO, String.format("%d: Completed in %dms", receivedTime, new Date().getTime() - time));
                super.onComplete();
            }

            @Override
            public void onReady() {
                time = new Date().getTime();
                super.onReady();
            }
            
            @Override
            public void onCancel() {
                logger.log(Level.INFO, String.format("%d: Call cancelled in %dms", receivedTime, new Date().getTime() - time));
                super.onCancel();
            }
        };
    }
}
```

You add an identifier `REQUEST_ID` to every request you receive and use this header to log any relevant information during the lifecycle of this request.
