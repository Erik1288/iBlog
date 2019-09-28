---
title: Pulsar-FAQ
date: 2019-04-29 14:48:18
tags:
---


### 这个bundle是什么意思？
One can either change the system default, or override it when creating a new namespace:

$ bin/pulsar-admin namespaces create my-tenant/my-namespace --clusters us-west --bundles 16

Copy
With this command, we're creating a namespace with 16 initial bundles. Therefore the topics for this namespaces can immediately be spread across up to 16 brokers.

In general, if the expected traffic and number of topics is known in advance, it's a good idea to start with a reasonable number of bundles instead of waiting for the system to auto-correct the distribution.

On a same note, it is normally beneficial to start with more bundles than number of brokers, primarily because of the hashing nature of the distribution of topics into bundles. For example, for a namespace with 1000 topics, using something like 64 bundles will achieve a good distribution of traffic across 16 brokers.

############# ThreadID:342 Time: 1559563265289 ################
java.lang.Thread Thread.java 1559 getStackTrace
-----------------------------------
org.apache.pulsar.debug.DebugUtil DebugUtil.java 15 commonStacktrace
-----------------------------------
org.apache.pulsar.debug.DebugUtil DebugUtil.java 11 stacktrace
-----------------------------------
org.apache.pulsar.broker.lookup.TopicLookupBase TopicLookupBase.java 280 lookupTopicAsync
-----------------------------------
org.apache.pulsar.broker.service.ServerCnx ServerCnx.java 263 lambda$handleLookup$3
-----------------------------------
java.util.concurrent.CompletableFuture CompletableFuture.java 602 uniApply
-----------------------------------
java.util.concurrent.CompletableFuture CompletableFuture.java 614 uniApplyStage
-----------------------------------
java.util.concurrent.CompletableFuture CompletableFuture.java 1983 thenApply
-----------------------------------
org.apache.pulsar.broker.service.ServerCnx ServerCnx.java 261 handleLookup
-----------------------------------
org.apache.pulsar.common.api.PulsarDecoder PulsarDecoder.java 112 channelRead
-----------------------------------
io.netty.channel.AbstractChannelHandlerContext AbstractChannelHandlerContext.java 362 invokeChannelRead
-----------------------------------
io.netty.channel.AbstractChannelHandlerContext AbstractChannelHandlerContext.java 348 invokeChannelRead
-----------------------------------
io.netty.channel.AbstractChannelHandlerContext AbstractChannelHandlerContext.java 340 fireChannelRead
-----------------------------------
io.netty.handler.codec.ByteToMessageDecoder ByteToMessageDecoder.java 323 fireChannelRead
-----------------------------------
io.netty.handler.codec.ByteToMessageDecoder ByteToMessageDecoder.java 297 channelRead
-----------------------------------
io.netty.channel.AbstractChannelHandlerContext AbstractChannelHandlerContext.java 362 invokeChannelRead
-----------------------------------
io.netty.channel.AbstractChannelHandlerContext AbstractChannelHandlerContext.java 348 invokeChannelRead
-----------------------------------
io.netty.channel.AbstractChannelHandlerContext AbstractChannelHandlerContext.java 340 fireChannelRead
-----------------------------------
io.netty.channel.DefaultChannelPipeline$HeadContext DefaultChannelPipeline.java 1434 channelRead
-----------------------------------
io.netty.channel.AbstractChannelHandlerContext AbstractChannelHandlerContext.java 362 invokeChannelRead
-----------------------------------
io.netty.channel.AbstractChannelHandlerContext AbstractChannelHandlerContext.java 348 invokeChannelRead
-----------------------------------
io.netty.channel.DefaultChannelPipeline DefaultChannelPipeline.java 965 fireChannelRead
-----------------------------------
io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe AbstractNioByteChannel.java 163 read
-----------------------------------
io.netty.channel.nio.NioEventLoop NioEventLoop.java 656 processSelectedKey
-----------------------------------
io.netty.channel.nio.NioEventLoop NioEventLoop.java 591 processSelectedKeysOptimized
-----------------------------------
io.netty.channel.nio.NioEventLoop NioEventLoop.java 508 processSelectedKeys
-----------------------------------
io.netty.channel.nio.NioEventLoop NioEventLoop.java 470 run
-----------------------------------
io.netty.util.concurrent.SingleThreadEventExecutor$5 SingleThreadEventExecutor.java 909 run
-----------------------------------
io.netty.util.concurrent.FastThreadLocalRunnable FastThreadLocalRunnable.java 30 run
-----------------------------------
java.lang.Thread Thread.java 748 run
-----------------------------------
#############################



```
/**
     *
     * Lookup broker-service address for a given namespace-bundle which contains given topic.
     *
     * a. Returns broker-address if namespace-bundle is already owned by any broker b. If current-broker receives
     * lookup-request and if it's not a leader then current broker redirects request to leader by returning
     * leader-service address. c. If current-broker is leader then it finds out least-loaded broker to own namespace
     * bundle and redirects request by returning least-loaded broker. d. If current-broker receives request to own the
     * namespace-bundle then it owns a bundle and returns success(connect) response to client.
     *
     * @param pulsarService
     * @param topicName
     * @param authoritative
     * @param clientAppId
     * @param requestId
     * @return
     */
    public static CompletableFuture<ByteBuf> lookupTopicAsync(PulsarService pulsarService, TopicName topicName,
        boolean authoritative, String clientAppId, AuthenticationDataSource authenticationData, long requestId) {

    final CompletableFuture<ByteBuf> validationFuture = new CompletableFuture<>();
    final CompletableFuture<ByteBuf> lookupfuture = new CompletableFuture<>();
    final String cluster = topicName.getCluster();

    // (1) validate cluster
    getClusterDataIfDifferentCluster(pulsarService, cluster, clientAppId).thenAccept(differentClusterData -> {

        if (differentClusterData != null) {
            if (log.isDebugEnabled()) {
                log.debug("[{}] Redirecting the lookup call to {}/{} cluster={}", clientAppId,
                        differentClusterData.getBrokerServiceUrl(), differentClusterData.getBrokerServiceUrlTls(),
                        cluster);
            }
            validationFuture.complete(newLookupResponse(differentClusterData.getBrokerServiceUrl(),
                    differentClusterData.getBrokerServiceUrlTls(), true, LookupType.Redirect, requestId, false));
        } else {
            // (2) authorize client
            try {
                checkAuthorization(pulsarService, topicName, clientAppId, authenticationData);
            } catch (RestException authException) {
                log.warn("Failed to authorized {} on cluster {}", clientAppId, topicName.toString());
                validationFuture.complete(newLookupErrorResponse(ServerError.AuthorizationError,
                        authException.getMessage(), requestId));
                return;
            } catch (Exception e) {
                log.warn("Unknown error while authorizing {} on cluster {}", clientAppId, topicName.toString());
                validationFuture.completeExceptionally(e);
                return;
            }
            // (3) validate global namespace
            checkLocalOrGetPeerReplicationCluster(pulsarService, topicName.getNamespaceObject())
                    .thenAccept(peerClusterData -> {
                        if (peerClusterData == null) {
                            // (4) all validation passed: initiate lookup
                            validationFuture.complete(null);
                            return;
                        }
                        // if peer-cluster-data is present it means namespace is owned by that peer-cluster and
                        // request should be redirect to the peer-cluster
                        if (StringUtils.isBlank(peerClusterData.getBrokerServiceUrl())
                                && StringUtils.isBlank(peerClusterData.getBrokerServiceUrl())) {
                            validationFuture.complete(newLookupErrorResponse(ServerError.MetadataError,
                                    "Redirected cluster's brokerService url is not configured", requestId));
                            return;
                        }
                        validationFuture.complete(newLookupResponse(peerClusterData.getBrokerServiceUrl(),
                                peerClusterData.getBrokerServiceUrlTls(), true, LookupType.Redirect, requestId,
                                false));

                    }).exceptionally(ex -> {
                        validationFuture.complete(
                                newLookupErrorResponse(ServerError.MetadataError, ex.getMessage(), requestId));
                        return null;
                    });
        }
    }).exceptionally(ex -> {
        validationFuture.completeExceptionally(ex);
        return null;
    });

    // Initiate lookup once validation completes
    validationFuture.thenAccept(validaitonFailureResponse -> {
        if (validaitonFailureResponse != null) {
            lookupfuture.complete(validaitonFailureResponse);
        } else {
            pulsarService.getNamespaceService().getBrokerServiceUrlAsync(topicName, authoritative)
                    .thenAccept(lookupResult -> {

                        if (log.isDebugEnabled()) {
                            log.debug("[{}] Lookup result {}", topicName.toString(), lookupResult);
                        }

                        if (!lookupResult.isPresent()) {
                            lookupfuture.complete(newLookupErrorResponse(ServerError.ServiceNotReady,
                                    "No broker was available to own " + topicName, requestId));
                            return;
                        }

                        LookupData lookupData = lookupResult.get().getLookupData();
                        if (lookupResult.get().isRedirect()) {
                            boolean newAuthoritative = isLeaderBroker(pulsarService);
                            lookupfuture.complete(
                                    newLookupResponse(lookupData.getBrokerUrl(), lookupData.getBrokerUrlTls(),
                                            newAuthoritative, LookupType.Redirect, requestId, false));
                        } else {
                            // When running in standalone mode we want to redirect the client through the service
                            // url, so that the advertised address configuration is not relevant anymore.
                            boolean redirectThroughServiceUrl = pulsarService.getConfiguration()
                                    .isRunningStandalone();

                            lookupfuture.complete(newLookupResponse(lookupData.getBrokerUrl(),
                                    lookupData.getBrokerUrlTls(), true /* authoritative */, LookupType.Connect,
                                    requestId, redirectThroughServiceUrl));
                        }
                    }).exceptionally(ex -> {
                        if (ex instanceof CompletionException && ex.getCause() instanceof IllegalStateException) {
                            log.info("Failed to lookup {} for topic {} with error {}", clientAppId,
                                    topicName.toString(), ex.getCause().getMessage());
                        } else {
                            log.warn("Failed to lookup {} for topic {} with error {}", clientAppId,
                                    topicName.toString(), ex.getMessage(), ex);
                        }
                        lookupfuture.complete(
                                newLookupErrorResponse(ServerError.ServiceNotReady, ex.getMessage(), requestId));
                        return null;
                    });
        }

    }).exceptionally(ex -> {
        if (ex instanceof CompletionException && ex.getCause() instanceof IllegalStateException) {
            log.info("Failed to lookup {} for topic {} with error {}", clientAppId, topicName.toString(),
                    ex.getCause().getMessage());
        } else {
            log.warn("Failed to lookup {} for topic {} with error {}", clientAppId, topicName.toString(),
                    ex.getMessage(), ex);
        }

        lookupfuture.complete(newLookupErrorResponse(ServerError.ServiceNotReady, ex.getMessage(), requestId));
        return null;
    });

    DebugUtil.stacktrace();

    return lookupfuture;
}
```