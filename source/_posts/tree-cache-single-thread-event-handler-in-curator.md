---
title: tree-cache-single-thread-event-handler-in-curator
date: 2017-09-20 15:32:22
tags: Curator
---


GenericFutureListener.operationComplete(Future) is directly called by an I/O thread. Therefore, performing a time consuming task or a blocking operation in the handler method can cause an unexpected pause during I/O. If you need to perform a blocking operation on I/O completion, try to execute the operation in a different thread using a thread pool.

``` java
channel.writeAndFlush(request).addListener(new ChannelFutureListener() {
    @Override
    public void operationComplete(ChannelFuture f) throws Exception {
        if (f.isSuccess()) {
            responseFuture.setSendRequestOK(true);
            return;
        } else {
            responseFuture.setSendRequestOK(false);
        }

        responseTable.remove(opaque);
        responseFuture.setCause(f.cause());
        responseFuture.putResponse(null);
        log.warn("send a request command to channel <" + addr + "> failed.");
    }
});
```