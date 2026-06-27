---
title: "fmsg: Structured Host-to-Host Messaging with Verifiable Threads"
category: info

docname: draft-mennell-art-fmsg-structured-messaging-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "ART"
keyword:
 - messaging
 - protocol
 - binary
venue:
  group: "Applications and Real-Time Area"
  type: "Area"
  mail: ""
  arch: ""
  github: "markmnl/fmsg-internet-draft"
  latest: "https://markmnl.github.io/fmsg-internet-draft/draft-mennell-art-fmsg-structured-messaging.html"

author:
 -
    fullname: "Mark Mennell"
    organization: Independent
    email: "markmnl@fmsg.io"

normative:

informative:
  FMSG-SPEC:
    title: "fmsg Protocol Specification"
    author:
      - name: Mark Mennell
    target: https://github.com/markmnl/fmsg
  FMSGD:
    title: "fmsgd Reference Host Implementation"
    author:
      - name: Mark Mennell
    target: https://github.com/markmnl/fmsgd

...

--- abstract

fmsg defines a message exchange protocol with structured binary messages where messages link to a previous message using a cryptographic hash, forming verifiable threads. Messages are sent via the sender's host to one or more recipient hosts. Receiving hosts verify origin IP address, message size, recipient policy and message ancestry before accepting message content. The protocol permits only the participants of a message to reply to it, and allows participants to add additional recipients after a message has been sent.

A distinguishing feature of fmsg is the receiving host can call back the sending host, during message exchange, to challenge for details of the message being sent. Processing such a challenge requires the sending host to be reachable and compute additional digests over the pending message.

This document describes the motivation and architecture of fmsg which is based on an existing experimental protocol specification and implementation. The purpose of this document is to solicit IETF venue and scope feedback before the full specification is split into applicable standards-track documents.



--- middle

# Introduction

Electronic messaging is dominated by two broad classes of systems. Internet email provides ownership and control at the domain level through open protocols, while modern messaging applications provide efficient conversational messaging but typically operate as closed services under a single provider.

This document introduces fmsg, an experimental message exchange protocol that combines ownership and control at the domain level with efficient conversational messaging. Messages are immutable binary objects exchanged between hosts responsible for the sender's and recipients' domains. Each message may reference a previous message using a cryptographic hash, forming a verifiable thread. Unlike conventional email, replies reference previous messages rather than reproducing earlier correspondence, allowing participants to verify message ancestry while transmitting only newly created message content.

The architecture of fmsg is based on a simple principle: the protocol defines only messages. There are no protocol-defined channels, rooms or groups; to receive a message, someone has to send it to you. Group conversations emerge naturally as participants reply to messages and add additional recipients over time.

The protocol incorporates sender verification and message integrity into the message exchange itself. Receiving hosts verify the origin of incoming messages, message ancestry and recipient policy before accepting message content.

This document describes the motivation, design goals and core architecture of fmsg. Detailed message formats, protocol procedures and transport bindings are intended to be specified separately. The purpose of this document is to introduce the architecture and solicit feedback from the IETF community regarding its applicability, scope and future standardisation.

# Terminology

This document uses the following terms:

Address: An fmsg address in the form @user@example.com identifying a recipient within a domain.

Client: An application that sends and receives messages through a Host.

Host: An implementation of the fmsg protocol responsible for sending and receiving messages to and from other fmsg hosts.

Sending Host: The Host transmitting a message.

Receiving Host: The Host receiving a message.

Message: An immutable unit of communication exchanged between Hosts.

Thread: A directed acyclic graph (DAG) of messages formed by references from each message to a previous message.

Recipient: An Address identified in a message as an intended recipient.

Sender: The Address originating a message.

Participants: The Sender and all Recipients of a message. Only Participants may reply to a message or add additional Recipients to a thread.

# Motivation

The design of fmsg was motivated by three primary objectives.

First, to provide the most efficient practical message exchange while preserving the capabilities that have made Internet email successful. In particular, fmsg retains unsolicited messaging, multiple recipients, arbitrary media types and ownership and control at the domain level, while avoiding the repeated transmission of information already known to participants in a conversation.

Second, to make conversation structure part of the protocol itself. Rather than relying on application-specific heuristics to reconstruct message threads, fmsg messages explicitly reference previous messages using cryptographic hashes. This allows participants to independently construct deterministic, verifiable message threads and enables consistent presentation of conversations across implementations.

Finally, to integrate sender verification and message integrity into the message exchange protocol. Rather than relying solely on complementary protocols or deployment-specific mechanisms, fmsg incorporates these properties into the architecture of message exchange itself. This is intended to simplify host deployment and operation, supporting a more decentralised set of messaging providers.


# Architectural Overview

The architecture of fmsg is centred on the exchange of immutable messages between hosts responsible for the sender's and recipients' domains. Each message may reference a previous message using a cryptographic hash, forming verifiable message threads. Group conversations emerge naturally as participants reply to messages and add additional recipients over time.

The architecture is guided by several core principles. Ownership and control remain at the domain level. Messages are immutable and exchanged independently between hosts. Message threads are deterministic and verifiable by all participants. Sender verification and message integrity are incorporated into the message exchange protocol. The protocol intentionally defines only the exchange of messages, leaving concerns such as user identity, authentication, message retrieval and client functionality undefined.

The following subsections describe these architectural principles and how they relate to one another.


## Ownership and Control

fmsg preserves ownership and control at the domain level. Every address belongs to a domain, and each domain is responsible for operating one or more Hosts authorised to send and receive messages for addresses within that domain. Messages are exchanged directly between Sending Hosts and Receiving Hosts responsible for the sender's and recipients' domains.

The core protocol defines only host-to-host message exchange. It does not define how users are authenticated, how addresses are provisioned, how messages are retrieved by clients, or how messages are stored within a domain. Thus allowing domain owners to retain control over their own infrastructure, operational policies and user management while interoperating through the fmsg message exchange protocol.


## Messages

The fundamental object in fmsg is the message.

Messages are immutable. Once created, the contents of a message never change. Corrections, replies, forwarding a message to additional Recipients and other conversation activity are represented by creating new messages rather than modifying existing ones. This immutability provides a stable foundation for verification, message integrity and conversation history.

Each message has exactly one Sender and one or more Recipients. Messages may contain arbitrary content represented by a Media Type and may include zero or more attachments. The protocol places no restrictions on the application semantics of the message content, allowing the exchange of plain text, rich text, images, audio, video or application-specific data.

Messages are exchanged independently between Sending Hosts and Receiving Hosts. The relationship between messages, including replies and conversation evolution, is described by the thread model rather than by the messages themselves.


## Threads

A Thread is formed by messages explicitly referencing previous messages. The first message in a Thread has no parent, while each subsequent message references the message to which it replies. A Thread is therefore an emergent protocol object; the protocol defines only messages and the relationships between them. As messages receive independent replies, a Thread naturally forms a directed acyclic graph (DAG).

By making message relationships explicit, every Participant can deterministically reconstruct the same Thread from the same set of messages. Replies reference an earlier message, allowing only newly created message content to be transmitted while preserving the context and ordering of the conversation.

The Thread model also provides a consistent basis for conversation policy. Because the Participants of a message are explicitly defined, a Receiving Host can determine whether a Sender is authorised to reply to a message and how additional Recipients become Participants in the Thread. The protocol therefore defines conversation evolution in terms of new immutable messages rather than modification of existing messages.

Finally, explicit Thread structure provides conversation context that is available to every Receiving Host. A Host can determine whether its local Participants have previously participated in a Thread and may use this information when making implementation-specific policy decisions, such as message classification, prioritisation or spam filtering.


## Participants

A Participant is the Sender or a Recipient of a Message.

Only a Participant of a Message can reply to that Message. This ensures that conversation participation is explicit and verifiable, and prevents unrelated parties from introducing messages into an existing Thread.

A Participant may add additional Recipients to a Thread by creating a new Message. Existing Messages are never modified. Instead, conversation membership evolves as new Messages are created, preserving the immutability of every Message while allowing conversations to naturally expand over time.

Because Participants are determined from the Thread itself, every Host can independently determine the Participants of any Message without requiring additional protocol state. This provides a consistent foundation for participant validation, conversation presentation and implementation-specific policy decisions.

## Hosts

A Host is responsible for sending and receiving Messages on behalf of a domain. Hosts exchange Messages directly with other Hosts using the fmsg protocol, providing interoperability while preserving ownership and control at the domain level.

By limiting the protocol boundary to host-to-host communication, fmsg enables independent implementations to interoperate while allowing each domain to retain complete control over its infrastructure, operational policies and internal message handling.

## Message Exchange

### Exchange Model

Message exchange occurs directly between a Sending Host and one or more Receiving Hosts. A Sending Host is responsible for transmitting a Message to each distinct Receiving Host by domain in the Message's Recipients. Each Receiving Host processes the Message independently and determines whether it will be accepted.

The message exchange protocol standardises communication between Sending Hosts and Receiving Hosts. Each Receiving Host independently validates and accepts or rejects a Message per Recipient according to the requirements of the protocol and its own local policies. Acceptance or rejection by one Receiving Host has no effect on the processing of the Message by any other Receiving Host.


### Message Acceptance

Message acceptance is based on three complementary validation domains: Sender validation, Message validation, and Recipient validation. A Receiving Host completes each validation domain before accepting a Message.

Sender validation establishes that the Sending Host is authorised to exchange Messages on behalf of the Sender's domain and, where required, that it possesses the Message being transmitted. This validation provides assurance that the Message originates from an authorised Host.

Message validation confirms the integrity and consistency of the Message itself. This includes validating message structure, size constraints, replay status, and the Message's ancestry within the Thread to ensure the Message can be incorporated into a consistent and verifiable conversation history.

Recipient validation determines whether each Recipient is willing to accept the Message. This includes validating recipient-specific policy, verifying that the Sender is permitted to communicate within the context of the Thread, and applying any implementation-specific acceptance policies. Because Recipient validation is performed independently by each Receiving Host, acceptance decisions remain under the control of the recipient's domain.

# Scope of this Document and Request for Feedback

This document deliberately describes only the motivation, design goals and core architecture of fmsg. It does not define message encodings, the on-the-wire protocol procedures, or transport bindings. Those details exist today in an experimental specification [FMSG-SPEC] and reference implementation [FMSGD], and are intended to be specified separately as the work matures.

The anticipated document structure separates the work into:

- this architectural overview;
- a message format document defining the binary message structure;
- a host-to-host protocol document defining the exchange procedures, including the challenge-response mechanism and message acceptance; and
- one or more transport binding documents (for example, a TCP and TLS binding).

The author is seeking feedback from the IETF community on the following questions, rather than a detailed review of the underlying specification at this stage:

1. Is the IETF, and specifically the Applications and Real-Time (ART) area, an appropriate venue for this work?
2. Is there interest in standardising a domain-level messaging protocol distinct from Internet email and from proprietary instant messaging systems?
3. Is the proposed scope appropriate, in particular the decision to standardise only host-to-host message exchange while leaving user identity, authentication, message retrieval and client behaviour out of scope?
4. Are there existing or in-progress efforts with which this work should be aligned or reconciled?

# Security Considerations

The fmsg architecture is designed around validation before acceptance. A Receiving Host validates a Message before accepting it into a Thread. This model ensures that sender verification, message integrity and recipient policy are evaluated before a Message becomes part of the recipient's conversation history.

Message acceptance comprises three validation domains: Sender validation, Message validation and Recipient validation. Together these establish that the Sending Host is authorised to send on behalf of the Sender's domain, that the Message is valid and consistent with the Thread, and that the Message satisfies recipient-specific policy.

The architecture provides explicit Thread relationships and immutable Messages, allowing Receiving Hosts to independently verify Thread ancestry and determine whether a Sender is authorised to participate in a conversation. Receiving Hosts may additionally perform protocol-defined challenge-response verification during message exchange to strengthen sender verification and message integrity.

The protocol standardises the information available to make message acceptance decisions while intentionally leaving operational policy to Receiving Hosts. Implementations may apply additional measures, including spam filtering, rate limiting and abuse detection, when determining whether to accept a Message.

Operational security considerations, transport security, denial-of-service mitigations and other implementation guidance are outside the scope of this document and are expected to be defined by the corresponding protocol specifications.

# Implementation Status

This document describes an architecture that has been implemented and evaluated through experimental software.

A complete protocol specification, including message definitions and protocol procedures, has been developed [FMSG-SPEC]. A canonical implementation following updates to the specification exists [FMSGD]. This implementation has been used to iterate development hand-in-hand with evolution of the specification.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

The author thanks the early implementers and reviewers of the fmsg specification for their feedback and contributions.

