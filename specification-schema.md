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
   
### 0.3.4 Queue
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
  - `Object` - Either a `String`, or a `Binary`, or a `KeyValue`, or a `Numeric`
  - `Numeric`:    
        - `Integer` - Integer in the range -(2^31) to 2^31 - 1 inclusive.  
        - `Long` - Integer in the range -(2^63) to 2^63 - 1 inclusive.  
        - `Float` - A 32-bit floating point number (binary32 [IEEE754](http://ieeexplore.ieee.org/servlet/opac?punumber=4610933)).  
        - `Double` - A 64-bit floating point number (binary64 [IEEE754](http://ieeexplore.ieee.org/servlet/opac?punumber=4610933)).  
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
   It will be use one of self-defining message types to encode the message body, and users are responsible for decode these bytes in a custom rules. 

### 2.2 Message Format

#### 2.2.1 Credential
   - Type: `KeyValue`  
   - Description: In order to   
   - Constraints: REQUIRED
   
#### 2.2.2 System Header
   - Type: `KeyValue`  
   - Description: All messages support the same set of header fields, and these header fields are used by system, which are usually used for such as identify and route messages.  
   - Constraints: REQUIRED
   
#### 2.2.3 User Header
   - Type: `KeyValue`  
   - Description: In addition to the system header, OMS provides a built-in user header for adding optional header fields to a message, and these attributes are represented as key-value forms.
   - Constraints: REQUIRED
   
#### 2.2.4 Message Body
   - Type: `Binary`  
   - Description: The field that contains user's business data.  
   - Constraints: OPTIONAL
   
### 2.3 Message System Header
#### 2.3.1 MessageId
   - Type: `String`  
   - Description: An unique identifier for a message.  When a message is sent, MessageId is ignored. When the send method returns it contains a provider-assigned value.
   - Constraints: REQUIRED and MUST be a non-empty string.
   
#### 2.3.2 Topic
   - Type: `String`   
   - Description: An identity of a message logic destination.
   - Constraints: REQUIRED and MUST be a non-empty string.
   
#### 2.3.3 Queue
   - Type: `String`   
   - Description: An identity of a message physical destination.
   - Constraints: REQUIRED and MUST be a non-empty string.
   
#### 2.3.4 BornTimestamp
   - Type: `Long` 
   - Description: The timestamp of the birth of the message. It is represented as a long value which is defined as the difference, measured in milliseconds, between this time and midnight, January 1, 1970 UTC.
   It is not the time the message was actually transmitted because the actual send may occur later due to transactions or other client side queueing of messages.  
   When a message is sent, BornTimestamp is ignored. When the send method returns, the field contains a time value somewhere in the interval between the call and the return.
   - Constraints: REQUIRED 
   
#### 2.3.5 BornHost
   - Type: `String` 
   - Description: When a message is sent, this field will be set with the local host info of producer.
   - Constraints: REQUIRED and MUST be a non-empty string.
   
#### 2.3.6 StoreTimestamp
   - Type: `Long` 
   - Description: The timestamp stored by server.  It is represented as a long value which is defined as the difference, measured in milliseconds, between this time and midnight, January 1, 1970 UTC.
   when a message is stored by server, this field will be set with the current timestamp of server.
   - Constraints: REQUIRED 
   
#### 2.3.7 StoreHost
   - Type: `String`
   - Description: The host info of the server that stores this message. when a message is stored by server, this field will be set with the host info of server.
   - Constraints: REQUIRED
   
#### 2.3.8 StartTime
   - Type: `Long` 
   - Description: The startup timestamp that a message can be delivered to consumer client. It is represented as a long value which is defined as the difference, measured in milliseconds, between this time and midnight, January 1, 1970 UTC.
   If this field isn't set explicitly, use `BornTimestamp` as the startup timestamp. 
   - Constraints: OPTIONAL
   
#### 2.3.9 StopTime
   - Type: `Long` 
   - Description: This field means that a message discarded time, if an undelivered message's stop time is reached, the message should be destroyed. If an earlier timestamp is set than `StartTime` or isn't set explicitly, that means the message does not expire.
   It is represented as a long value which is defined as the difference, measured in milliseconds, between this time and midnight, January 1, 1970 UTC.
   - Constraints: OPTIONAL
   
#### 2.3.10 Timeout
   - Type: `Long` 
   - Description: It represents a message time-to-live value. If the this field is specified as zero, that indicates the message does not expire, and this field has higher priority than START_TIME/STOP_TIME header fields.
   It is represented as a long value which is defined as the difference, measured in milliseconds, between this time and midnight, January 1, 1970 UTC.
   - Constraints: OPTIONAL
   
#### 2.3.11 Priority
   - Type: `Integer`
   - Description: OMS defines a ten level priority value with 1 as the lowest priority and 10 as the highest, and the default priority is 5. The priority beyond this region will be ignored.
   OMS does not require or provide any guarantee that the message should be delivered  in priority order strictly, but the vendor should provide a best effort to deliver expedited messages ahead of normal messages.
   - Constraints: OPTIONAL
   
#### 2.3.12 Reliability
   - Type: `Integer`
   - Description: OMS defines two modes of message delivery:  
   **PERSISTENT**: when this field value is set with 0, the persistent mode instructs the vendor should provide stable storage to ensure the message won't be lost.  
   **NON_PERSISTENT**: when this field value is set with 1, this mode does not require the message be logged to stable storage, in most cases, the memory storage is enough for better performance and lower cost.  
   - Constraints: OPTIONAL
   
#### 2.3.13 SearchKey
   - Type: `String`
   - Description: The keyword indexes will be built by the search keys, users can query similar messages through these indexes and have a quick response.
   - Constraints: OPTIONAL

#### 2.3.14 ScheduleExpression
   - Type: `String`
   - Description: The message will be delivered by the specified SCHEDULE_EXPRESSION, which is a [CRON expression](https://en.wikipedia.org/wiki/Cron#CRON_expression).
   When this filed isn't set explicitly, this means this message should be delivered immediately.
   - Constraints: OPTIONAL
   
#### 2.3.15 TraceId
   - Type: `String`
   - Description: This identifier represents a global and unique identification, to associate key events in the whole lifecycle of a message,
   like sent by who, stored at where, and received by who. And, the messaging system only plays exchange role in a distributed system in most cases,
   so the TraceID can be used to trace the whole call link with other parts in the whole system.
   - Constraints: OPTIONAL
   
#### 2.3.16 CompressionLevel
   - Type: `Integer`
   - Description: This field represents the message body compress level, 0 represents uncompress, 
   but vendors are free to choose the compression algorithm and define compression levels, but they must ensure that the decompressed message is delivered to the user.
   - Constraints: OPTIONAL

#### 2.3.17 TransactionId
   - Type: `String`
   - Description: This field is used in transactional message, and it can be used to trace a transaction, 
   so the same `TransactionId` will be appeared not only in prepare message, but also in commit message, and consumer received message also contains this field.
   - Constraints: OPTIONAL


### Notational Conventions
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to
be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).
## Appendix 
### Example of OpenMessaging API
### Change History
