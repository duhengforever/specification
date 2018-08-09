# OpenMessaging Specification

## Table of Contents
- [License](#license)
- [Overview](#0-overview)
- [Type System](#1-type-system)
- [Message Model](#2-message-model)
## License
## 0 Overview
### 0.1 What is OpenMessaging?
   OpenMessaging is a cloud-oriented, vendor-neutral open standard for distributed messaging.
### 0.2 Why OpenMessaging?
   Messaging and Streaming products have been widely used in modern architecture and data processing, for decoupling, 
   queuing, buffering, ordering, replicating, etc. But when data transfers across different messaging and streaming platforms, 
   the compatibility problem arises, which always means much additional work. Although JMS was a good solution during the past 
   decade, it is limited in java environment, lacks specified guidelines for load balance/fault-tolerance, administration, 
   security, and streaming feature, which make it not good at satisfying modern cloud-oriented messaging and streaming applications.
### 0.3 OpenMessaging Terminologies
#### 0.3.1 Topic
   An administered object that encapsulates the identity of a message destination for messaging.
   
#### 0.3.2 Producer
   An object that sending a message to all consumers of a topic.
   
#### 0.3.3 Consumer
   An object that is used for receiving messages sent to a topic.
   
#### 0.3.4 Queue
   An administered object that encapsulates the identity of a message destination.
   
#### 0.3.5 Delivery Semantics
   **At least once**: a message will be consumed at least once.  
   **At most once**: a message will be consumed at most once, in this semantics, messages may be lost.  
   **Exactly once**: a message will be consumed once and only once.  
#### 0.3.6 Queue Ordering
#### 0.3.7 Durability
### 0.4 Difference between AMQP, COBAR & JMS

## 1 Type System
  The following abstract data types are available for use in attributes.
  
  - `String` - Sequence of printable Unicode characters.
  - `Binary` - Sequence of bytes.
  - `KeyValue` - `String`-indexed dictionary of `Object`-typed values
  - `Numeric`:    
        - `Short` - Integer in the range -(2^15) to 2^15 - 1 inclusive.
        - `Integer` - Integer in the range -(2^31) to 2^31 - 1 inclusive.  
        - `Long` - Integer in the range -(2^63) to 2^63 - 1 inclusive.  
        - `Float` - A 32-bit floating point number (binary32 [IEEE754](http://ieeexplore.ieee.org/servlet/opac?punumber=4610933)).  
        - `Double` - A 64-bit floating point number (binary64 [IEEE754](http://ieeexplore.ieee.org/servlet/opac?punumber=4610933)).  
  - `Object` - Either a `String`, or a `Binary`, or a `KeyValue`, or a `Numeric`
  - `URI` - String expression conforming to `URI-reference`
    as defined in
    [RFC 3986 §4.1](https://tools.ietf.org/html/rfc3986#section-4.1).
    
  The `Object` type is a variant type that can take the shape of either a
  `String` or a `Binary` or a `KeyValue` or a `Numeric`. The type system is intentionally
  abstract, and therefore it is left to implementations how to represent the
  variant type.
 
## 2 Message Model
### 2.1 Message Type
#### 2.1.1 Bytes Message
   A message that whose body contains a stream of uninterpreted bytes. This message type is for literally encoding a body to match an existing message format.
   It will be use one of self-defining message types to encode the message body, and vendors are responsible for decode these bytes in a custom rules. 

### 2.2 Message Format
   In the OpenMessaging, a message consists of five parts: the version, the credential, the system header, the user header and the message body.
#### 2.2.1 version
   - Type: `String`  
   - Description: The version of this message schema. 
   - Constraints: REQUIRED 
   
#### 2.2.2 credential
   - Type: `KeyValue`  
   - Description: A [credential](https://en.wikipedia.org/wiki/Credential) is an attestation of qualification, competence, or authority issued to an individual by a third party with a relevant or authority or assumed competence to do so.  
   In OpenMessaging, it's represented the authority to receive a message.
   - Constraints: OPTIONAL
   
#### 2.2.3 systemHeader
   - Type: `KeyValue`  
   - Description: All messages support the same set of header fields, and these header fields are used by system, which are usually used for such as identify and route messages.
    Specific fields can be found in the next [chapter](#2.3-message-system-header).
   - Constraints: REQUIRED
   
#### 2.2.4 userHeader
   - Type: `KeyValue`  
   - Description: In addition to the system header, OMS provides a built-in user header for adding optional header fields to a message, and these attributes are represented as key-value forms.
   - Constraints: REQUIRED
   
#### 2.2.5 body
   - Type: `Binary`  
   - Description: This field is the part of transmitted data that is the actual intended message contains application data.   
   The message body is completely transparent to the server, the server cannot view or modify the message body.  
   - Constraints: OPTIONAL
   
### 2.3 Message System Header
#### 2.3.1 messageId
   - Type: `String`  
   - Description: An unique identifier for a message.  When a message is sent, MessageId is ignored. When the send method returns it contains a provider-assigned value.
   - Constraints: REQUIRED and MUST be a non-empty `String`.
   
#### 2.3.2 topic
   - Type: `String`   
   - Description: An identity of a message logic destination.
   - Constraints: REQUIRED and MUST be a non-empty `String`.
   
#### 2.3.3 queue
   - Type: `String`   
   - Description: An identity of a message physical destination.
   - Constraints: REQUIRED and MUST be a non-empty `String`.
   
#### 2.3.4 bornTimestamp
   - Type: `Long` 
   - Description: The timestamp of the birth of the message.  
   It is not the time the message was actually transmitted because the actual send may occur later due to transactions or other client side queueing of messages.   
   When a message is sent, BornTimestamp is ignored. When the send method returns, the field contains a time value somewhere in the interval between the call and the return.   
   It is represented as a long value which is defined as the difference, measured in milliseconds, between this time and midnight, January 1, 1970 UTC.
   - Constraints: REQUIRED 
   
#### 2.3.5 bornHost
   - Type: `String` 
   - Description: When a message is sent, this field will be set with the local host info of producer.
   - Constraints: REQUIRED and MUST be a non-empty `String`.
   
#### 2.3.6 storeTimestamp
   - Type: `Long` 
   - Description: The timestamp stored by server. when a message is stored by server, this field will be set with the current timestamp of server.  
   It is represented as a long value which is defined as the difference, measured in milliseconds, between this time and midnight, January 1, 1970 UTC.
   - Constraints: REQUIRED 
   
#### 2.3.7 storeHost
   - Type: `String`
   - Description: The host info of the server that stores this message. when a message is stored by server, this field will be set with the host info of server.
   - Constraints: REQUIRED
   
#### 2.3.8 startTime
   - Type: `Long` 
   - Description: This field represents the start timestamp of which the message can be delivered to consumer.   
   If this field isn't set explicitly, use `bornTimestamp` as the startup timestamp.  
   It is represented as a long value which is defined as the difference, measured in milliseconds, between this time and midnight, January 1, 1970 UTC.
   - Constraints: OPTIONAL
   
#### 2.3.9 stopTime
   - Type: `Long` 
   - Description: This field represents the discard time of the message, if an undelivered message's stop time is reached, the message should be destroyed. If an earlier timestamp is set than `startTime` or isn't set explicitly, that means the message does not expire.  
   It is represented as a long value which is defined as the difference, measured in milliseconds, between this time and midnight, January 1, 1970 UTC.
   - Constraints: OPTIONAL
   
#### 2.3.10 timeout
   - Type: `Long` 
   - Description: It represents a message time-to-live value. If the this field is specified as zero, that indicates the message does not expire, and this field has higher priority than `startTime/stopTime` header fields.  
   Compared with the field `stopTime`, this field represents the relative time from the `startTime` to the present, when the clock is not synchronized in the distributed system, this field is used to control whether a message should be expires.    
   - Constraints: OPTIONAL
   
   
#### 2.3.11 priority
   - Type: `Integer`
   - Description: OMS defines a ten level priority value with 1 as the lowest priority and 10 as the highest, and the default priority is 5. The priority beyond this region will be ignored.
   OMS does not require or provide any guarantee that the message should be delivered  in priority order strictly, but the vendor should provide a best effort to deliver expedited messages ahead of normal messages.
   - Constraints: OPTIONAL
   
#### 2.3.12 reliability
   - Type: `Integer`
   - Description: OMS defines two modes of message delivery:  
   **PERSISTENT**: when this field value is set with 0, the persistent mode instructs the vendor should provide stable storage to ensure the message won't be lost.  
   **NON_PERSISTENT**: when this field value is set with 1, this mode does not require the message be logged to stable storage, in most cases, the memory storage is enough for better performance and lower cost.  
   - Constraints: OPTIONAL
   
#### 2.3.13 searchKey
   - Type: `String`
   - Description: The keyword indexes will be built by the search keys, users can query similar messages through these indexes and have a quick response.
   - Constraints: OPTIONAL

#### 2.3.14 scheduleExpression
   - Type: `String`
   - Description: The message will be delivered by the specified SCHEDULE_EXPRESSION, which is a [CRON expression](https://en.wikipedia.org/wiki/Cron#CRON_expression).
   When this filed isn't set explicitly, this means this message should be delivered immediately.
   - Constraints: OPTIONAL
   
#### 2.3.15 traceId
   - Type: `String`
   - Description: This identifier represents a global and unique identification, to associate key events in the whole lifecycle of a message,
   like sent by who, stored at where, and received by who. And, the messaging system only plays exchange role in a distributed system in most cases,
   so the TraceID can be used to trace the whole call link with other parts in the whole system.
   - Constraints: OPTIONAL
   
#### 2.3.16 compressionLevel
   - Type: `Integer`
   - Description: This field represents the message body compress level, 0 represents uncompress, 
   but vendors are free to choose the compression algorithm and define compression levels, but they must ensure that the decompressed message is delivered to the user.
   - Constraints: OPTIONAL

#### 2.3.17 transactionId
   - Type: `String`
   - Description: This field is used in transactional message, and it can be used to trace a transaction, 
   so the same `transactionId` will be appeared not only in prepare message, but also in commit message, and consumer received message also contains this field.
   - Constraints: OPTIONAL
   
### 2.4 Message  Credential 
#### 2.4.1 accountId
   - Type: `String`  
   - Description: 
   - Constraints: OPTIONAL
#### 2.4.2 accessKey
   - Type: `String`  
   - Description: 
   - Constraints: OPTIONAL
   
### Notational Conventions
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to
be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

## Appendix 
### Example of OpenMessaging API
```json
{
    "messages": {
        "message": {
           "version":"0.0.1",
           "credential":{
               "accountId":"123456789",
               "accessKey":"123456789"
           },
           "sysHeaders": {
               "messageId": "1234-123-456",
               "topic":"test",
               "queue":1,
               "bornTimestamp": 123456789012,
               "bornHost": "172.24.0.101",
               "storeTimestamp": 123456789013,
               "storeHost": "172.24.0.102",
               "timeout": 123456789999,
               "priority": 1,
               "traceId": "1234-123456-1",
               "transactionId": "5566-123456-1",
               "searchKey": "hello",
               "reliability":1
            },
            "userHeaders": {
               "service":"helloService"
            },
            "bodies": {}
        }
    }
}
```
### Change History
