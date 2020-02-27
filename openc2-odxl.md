## 1.7 Suitability
This OpenC2 over OpenDXL transfer specification 
is suitable for operational environments where: 

* Connectivity between OpenC2 Producers and OpenC2 Consumers is: 
  * Highly available, with infrequent network outages
  * Of sufficient bandwidth that no appreciable message delays or dropped packets are experienced 
* In-band negotiation of a connection initiated by either Producer or Consumer is possible without 
requiring an out-of-band signaling network.

An additional application for this transfer specification is interoperability test environments.

# 2 Operating Model

This section describes the operating model used when transferring OpenC2 Commands and Responses using OpenDXL

## 2.1: Endpoint definitions
Each endpoint of an OpenC2-over-OpenDXL interaction has an OpenC2 role and at least one OpenDXL function. 

OpenC2 Consumers will necessarily be OpenDXL Services, a service being as defined by the OpenDXL specification 
OpenC2 Producers will be OpenDXL clients as defined by the OpenDXL specification. OpenC2 producers, should at
a minimum, publish messages to the service, and optionally, subscribe to responses sent by the service


Figure 2 illustrates the Producer / Consumer interactions. 

***Need a different pub sub diagram here***


---
# 3 Protocol Mappings
The section defines the requirements for using OpenDXL and TLS with OpenC2, including general requirements and protocol mappings for the operating configuration described in Section 2.

## 3.1 Layering Overview
When using OpenDXL for OpenC2 Message transfer, the layering model is:

| Layer | Description |
|:---|:---|
| OpenC2 Content | The OpenC2 Language Specification defines the overall OpenC2 language, and the Actuator Profile(s) implemented by any particular endpoint scopes the OpenC2 actions, targets, arguments, and specifiers that apply when commanding that type of Actuator.  |
| Serialization | Serialization converts internal representations of OpenC2 content into a form that can be transmitted and received. The OpenC2 default serialization is JSON. |
| Message | The message layer provides a content- and transport-independent mechanism for conveying requests and responses.  A Message consists of content plus a set of meta items such as content type and version, sender, timestamp, and correlation id.  This layer maps the transport-independent definition of each message element to its transport-specific on-the-wire representation.|
| OpenDXL | The OpenDXL layer is responsible for conveying request and response Messages, as described in this specification. |
| MQTT  | The MQTT layer is responsible for connection management, and message delivery|
| TLS | The TLS layer is responsible for authentication of connection endpoints and confidentiality and integrity of transferred Messages.  |
| Lower Layer Transport | The lower protocol layers are responsible for end-to-end delivery of Messages. TCP/IP is the most common suite of lower layer protocols used with OpenDXL. |

## 3.2 General Requirements
This section defines serialization, OpenDXL, and TLS requirements.

### 3.2.1 Serialization and Content Types
While the OpenC2 language is agnostic of serialization, when transferring OpenC2 Messages over OpenDXL/TLS as described in this specification, the default JSON serialization described in [[OpenC2-Lang-v1.0](#openc2-lang-v10)] MUST be supported.

As described in [OpenC2-Lang-v1.0], transfer protocols must convey message elements. This is specified via OpenDXL topic hierarchy as below

Topic : `/openc2/{version}/`
More specifically, for version 1.0 of the openc2 language standards, v1_0 shall be used as version. Openc2 messages shall be published to topics starting with /openc2/v1_0

Unless otherwise mentioned in the message definition, the default format for messages over OpenDXL shall be JSON. For any other serialization format, the field "content-type" of OpenDXL messaging MUST indicate the serialization used for the message. The specification does not mandate a serialization format as long as the consumer is able to successfully interpret the the serialization identifier.

OpenC2 communications over OpenDXL must use OpenDXL message version 3 or higher

### 3.2.2 OpenDXL Usage
OpenC2 Consumers MUST be OpenDXL services, Openc2 producers MUST NOT manifest as an OpenDXL service. OpenC2 producers must otherwise be valid OpenDXL clients, connected to the OpenDXL fabric via the mechanisms specified by the OpenDXL specifications.

The above doesn’t not preclude the ability for a single entity to function both as an openC2 producer as well as an openC2 consumer. The mapping of openc2 consumer to OpenDXL service, and of openc2 producer to an OpenDXL client is a purely logical construct

:Operating Model 


**Table 3-1: OpenDXL Message Use**

Each OpenDXL message MUST contain only a single OpenC2 Command or Response message. This does not preclude a Producer and Consumer exchanging multiple OpenC2 Command and Response Messages over time over a given set of topics. Depending on the set-up, a publisher and subscriber can have multiple connections, but a sequence of OpenC2 interactions can spread over multiple connections. In some cases the connection may drop, but the session remains open (in an idle state).

The OpenDXL field "message_id" SHOULD be populated with the request_id string supplied by the Producer.

### 3.2.3 TLS Usage
OpenDXL, transmits messages over TLS, is specified in Section 2 of [[RFC2818](#rfc2818)]. OpenC2 endpoints MUST accept TLS version 1.2 [[RFC5246](#rfc5246)] connections or higher for confidentiality, identification, and authentication when sending OpenC2 Messages over OpenDXL, and SHOULD accept TLS Version 1.3 [[RFC8446](#rfc8446)] or higher connections.

OpenC2 endpoints MUST NOT support any version of TLS prior to v1.2 and MUST NOT support any version of Secure Sockets Layer (SSL). 

The implementation and use of TLS SHOULD align with the best currently available security guidance, such as that provided in [[RFC7525](#rfc7525)]/BCP 195.

The TLS session MUST use non-NULL ciphersuites for authentication, integrity, and confidentiality.  Sessions MAY be renegotiated within these constraints.

OpenC2 endpoints supporting TLS v1.2 MUST NOT use any of the blacklisted ciphersuites identified in Appendix A of [[RFC7540](#rfc7540)]. 

OpenC2 endpoints supporting TLS 1.3 MUST NOT implement zero round trip time resumption (0-RTT).

### 3.2.4 Authentication

OpenC2 producers and consumers do not authenticate or authorize each other directly. Instead, authentication and authorization between OpenC2 producers and consumers is controlled via access policies configured at the OpenDXL broker. The OpenDXL broker manages access to topics, and thus controls the traffic between producers and consumers. 


## 3.3 Mapping OpenC2 constructs to  OpenDXL

### 3.3.1 OpenC2 Publisher as a DXL Client: 
This section defines OpenDXL requirements that apply when the OpenC2 producer is a OpenDXL client. An OpenC2 producer may transfer messages over one of the two OpenDXL transport mechanisms
	
	
	A. Event Messages : Messages for which the producer does 
	not expect a response of any form back from the consumer. 
	Event messages have the following fields :
		a. destination_topic :  The service topic of an 
		OpenC2 that producers intends to communicate to
		b. message_type : 2 (MESSAGE_TYPE_EVENT)
		c. message_id : A GUID denoting the id of the message
		d. content_type : Define the serialization format used.
		Default to JSON
		e. payload :  Serialized data containing the openc2 command 
		f. version :  OpenDXL message protocol version. 
		MUST be 3 or greater
		g. created : Timestamp in milliseconds since epoch
		
		
	B. Request Messages : Messages for which the producer 
	expects a response back from the consumer. Request 
	messages are an extension  of Event messages and carry 
	the same message fields as event messages, other than 
	the following differences:
		A. message_type : 0 (MESSAGE_TYPE_REQUEST)
		B. reply_to_topic : Openc2 producers that wish to 
		process responses asynchronously may setup a dedicated 
		and unique response topic and subscribe to it. 
		This fields informs the OpenC2 consumers about 
		the topic the responses should be sent to.

OpenC2 publisher messages over OpenDXL conform to OpenC2 language specification as below:

| OpenC2 Spec | OpenDXL mapping |
| --- | --- |
| Content | Payload |
| Content_type / Msg_type | Split between topic name ( `/openc2/{version}`), and content_type of command/{format} | 
| Status | Populated in response |
| Request_id | Message_id
|Created | Created , automatically populated |
| From | Combination of Source_client_id, source_broker_id, source_tenant_id |
| To | Combination of Service_id , service name and service topic |

### 3.3.2 OpenC2 Consumer as OpenDXL service
This section defines OpenDXL requirements that apply when the OpenC2 Consumer is an OpenDXL service

An openC2 consumer over OpenDXL must adhere to OpenDXL specification for a "service". The service must be named. The service name must be an alphanumeric string. 
The service MUST expose at least one topic. Topic names MUST be alphanumeric  and MUST begin with 
`/openc2/{version}/`

* Topic names are case sensitive
* Topic name must not  exceed 65536 characters. 
* A consumer may expose an arbitrary number of topics

The following message fields must be populated when transferring open c2 commands to a consumer:
```
A. destination_topic : This is the topic that the consumer 
is active on. The topic name MUST begin with 
/openc2/{version}/commands/*

B. message_type : The numeric type of the message as defined 
by OpenDXL. 
   a. MESSAGE_TYPE_REQUEST , with a numeric value of 0 : This 
   message type MUST be used when the openC2 producer is 
   expecting a response back from the consumer
   b. MESSAGE_TYPE_EVENT , with a numeric value of 2. This message 
   type MUST be used when the openc2 producer is NOT expecting 
   any response from the consumer

C. content-type : The serialization formation used by the message. 
Defaults to JSON.

D. message_id : An unique id associated with the message

E. payload : Serialized data containing the openc2 message : Maps
to "content" message element 

F. version : Version of DXL message specification. 
The minimum version required for openc2 over OpenDXL is version 3

G. source_client_id : Identity of the client the message is from.
This field is automatically populated by the OpenDXL broker. 
Corresponds to the "from" field of the message

H. created:  Milliseconds since epoch

I. Destination topic covers the to: field
```