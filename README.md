# How to reproduce kafka credential-free token-refresh issue

1. Run commands of `az login`
2. Run the application via Intellij
3. Watch the log from class of `org.apache.kafka.common.security.oauthbearer.internals.expiring.ExpiringCredentialRefreshingLogin` to see when it will start to execute token-refreshing. There should be logs like
    ```log
    2022-09-01 17:25:23.719  INFO 105268 --- [xxx] .o.i.e.ExpiringCredentialRefreshingLogin : [Principal=:xxxx@microsoft.com]: Expiring credential re-login sleeping until: Thu Sep 01 18:28:54 CST 2022
    ```

Kafka will start to try re-login before token-refreshing and will stop until a new token is fetched.

4. After the token is refreshed, run the command to send a message immediately:
    ```shell
    curl -X POST http://localhost:8080/messages?message=hello
    ```
    
    Then the below exception should be able to appear.
    
    ```log
    org.apache.kafka.common.errors.SaslAuthenticationException: An error: (java.security.PrivilegedActionException: javax.security.sasl.SaslException: No OAuth Bearer tokens in Subject's private credentials [Caused by java.io.IOException: No OAuth Bearer tokens in Subject's private credentials]) occurred when evaluating SASL token received from the Kafka Broker. Kafka Client will go to AUTHENTICATION_FAILED state. Caused by: javax.security.sasl.SaslException: No OAuth Bearer tokens in Subject's private credentials at org.apache.kafka.common.security.oauthbearer.internals.OAuthBearerSaslClient.evaluateChallenge(OAuthBearerSaslClient.java:120) at org.apache.kafka.common.security.authenticator.SaslClientAuthenticator.lambda$createSaslToken$1(SaslClientAuthenticator.java:534) at java.base/java.security.AccessController.doPrivileged(AccessController.java:712) at java.base/javax.security.auth.Subject.doAs(Subject.java:439) at org.apache.kafka.common.security.authenticator.SaslClientAuthenticator.createSaslToken(SaslClientAuthenticator.java:534) at org.apache.kafka.common.security.authenticator.SaslClientAuthenticator.sendSaslClientToken(SaslClientAuthenticator.java:433) at org.apache.kafka.common.security.authenticator.SaslClientAuthenticator.sendInitialToken(SaslClientAuthenticator.java:332) at org.apache.kafka.common.security.authenticator.SaslClientAuthenticator.authenticate(SaslClientAuthenticator.java:298) at org.apache.kafka.common.network.KafkaChannel.prepare(KafkaChannel.java:181) at org.apache.kafka.common.network.Selector.pollSelectionKeys(Selector.java:543) at org.apache.kafka.common.network.Selector.poll(Selector.java:481) at org.apache.kafka.clients.NetworkClient.poll(NetworkClient.java:560) at org.apache.kafka.clients.consumer.internals.ConsumerNetworkClient.poll(ConsumerNetworkClient.java:265) at org.apache.kafka.clients.consumer.internals.ConsumerNetworkClient.poll(ConsumerNetworkClient.java:236) at org.apache.kafka.clients.consumer.KafkaConsumer.pollForFetches(KafkaConsumer.java:1297) at org.apache.kafka.clients.consumer.KafkaConsumer.poll(KafkaConsumer.java:1238) at org.apache.kafka.clients.consumer.KafkaConsumer.poll(KafkaConsumer.java:1211) at jdk.internal.reflect.GeneratedMethodAccessor98.invoke(Unknown Source) at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) at java.base/java.lang.reflect.Method.invoke(Method.java:568) at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:344) at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:208) at jdk.proxy2/jdk.proxy2.$Proxy129.poll(Unknown Source) at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.pollConsumer(KafkaMessageListenerContainer.java:1529) at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doPoll(KafkaMessageListenerContainer.java:1519) at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.pollAndInvoke(KafkaMessageListenerContainer.java:1343) at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.run(KafkaMessageListenerContainer.java:1255) at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:539) at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264) at java.base/java.lang.Thread.run(Thread.java:833) Caused by: java.io.IOException: No OAuth Bearer tokens in Subject's private credentials at org.apache.kafka.common.security.oauthbearer.internals.OAuthBearerSaslClientCallbackHandler.handleCallback(OAuthBearerSaslClientCallbackHandler.java:104) at org.apache.kafka.common.security.oauthbearer.internals.OAuthBearerSaslClientCallbackHandler.handle(OAuthBearerSaslClientCallbackHandler.java:83) at org.apache.kafka.common.security.oauthbearer.internals.OAuthBearerSaslClient.evaluateChallenge(OAuthBearerSaslClient.java:92) ... 29 common frames omitted
    ```