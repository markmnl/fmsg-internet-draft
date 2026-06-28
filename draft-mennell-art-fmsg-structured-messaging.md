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
  mail: "dispatch@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/dispatch/"
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

fmsg is a message exchange protocol between domain hosts. Messages are structured binary objects forming threads where participation is required to reply. Additional recipients can be added to messages after they have been sent. Sender verification and message integrity guarantees are built into the protocol itself.

This document describes the motivation and architecture of fmsg, an existing protocol specification and implementation are referenced but the full specification is not included here. The purpose of this document is to solicit IETF feedback on venue and scope before/if the specification can develop into standards-track documents.


--- middle

# Introduction

This document introduces fmsg, an open message exchange protocol that provides ownership and control at the domain level together with efficient conversational messaging. Messages are immutable, structured binary objects with reference to a parent message they are in reply to. Threads of messages form by following their parent reference. Hosts verify sender origin, sender was a participant of parent message, message size, and recipient policy before downloading message content. During message exchange receiving hosts can actively call back the sending host to challenge for details of the message being sent. Messages can be accepted/rejected for each recipient belonging to a receiving host.

Much of the architecture of fmsg is based on: fmsg is just messages. Unlike many popular group based messaging platforms, there are no protocol-defined channels, rooms or groups; to receive a message, someone has to send it to you. Group conversations emerge naturally as participants reply to messages and add additional recipients over time.


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

Second, to make conversation structure part of the protocol itself. Rather than relying on application-specific heuristics to reconstruct message threads, fmsg messages explicitly reference previous messages using cryptographic hashes. This allows participants to independently construct deterministic, verifiable message threads and enables 1) verify prior participants, and; 2) consistent presentation of conversations across implementations.

Finally, to integrate sender verification and message integrity into the message exchange protocol. Rather than relying solely on complementary protocols or deployment-specific mechanisms, fmsg incorporates these properties into the architecture of message exchange itself. This is intended to simplify host deployment and operation, supporting a more decentralised set of messaging providers.


# Architectural Overview

This section describes the architecture of fmsg in terms of its core objects (messages, threads, participants and hosts) and the message exchange between hosts that ties them together.

The architecture is guided by several core principles. Ownership and control remain at the domain level. Messages are immutable and exchanged independently between hosts. Message threads are deterministic and verifiable by all participants. Sender verification and message integrity are incorporated into the message exchange protocol. The protocol intentionally defines only the exchange of messages, leaving concerns such as user identity, authentication, message retrieval and client functionality undefined.

The following subsections describe these principles and how they relate to one another.


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

A Host is an implementation of the fmsg protocol that sends and receives Messages on behalf of a domain. A domain may operate more than one Host, and a Host may serve more than one domain. Hosts are the only protocol-visible actors in fmsg: clients, user identity and message storage sit behind a Host and are not exposed by the message exchange protocol.

A Message exchange always takes place between a Sending Host and a Receiving Host. These roles are assigned per Message: a Host acts as a Sending Host when transmitting a Message and as a Receiving Host when accepting one.

## Message Exchange

### Exchange Model

Message exchange occurs directly between a Sending Host and one or more Receiving Hosts. A Sending Host is responsible for transmitting a Message to each distinct Receiving Host by domain in the Message's Recipients. Each Receiving Host processes the Message independently and determines whether it will be accepted.

The message exchange protocol standardises communication between Sending Hosts and Receiving Hosts. Each Receiving Host independently validates and accepts or rejects a Message per Recipient according to the requirements of the protocol and its own local policies. Acceptance or rejection by one Receiving Host has no effect on the processing of the Message by any other Receiving Host.


### Message Acceptance

Message acceptance is based on three complementary validation stages: Sender validation, Message validation, and Recipient validation. A Receiving Host completes each validation stage before accepting a Message.

Sender validation establishes that the Sending Host is authorised to exchange Messages on behalf of the Sender's domain and, where required, that it possesses the Message being transmitted. This validation provides assurance that the Message originates from an authorised Host.

Message validation confirms the integrity and consistency of the Message itself. This includes validating message structure, size constraints, replay status, and the Message's ancestry within the Thread to ensure the Message can be incorporated into a consistent and verifiable conversation history.

Recipient validation determines whether each Recipient is willing to accept the Message. This includes validating recipient-specific policy, verifying that the Sender is permitted to communicate within the context of the Thread, and applying any implementation-specific acceptance policies. Because Recipient validation is performed independently by each Receiving Host, acceptance decisions remain under the control of the recipient's domain.

### The Challenge

A distinguishing feature of fmsg is the automatic challenge. While a Message is being received, and before its content is accepted, a Receiving Host may open a separate connection back to the Sending Host and challenge it for details of the Message currently being transmitted. The Sending Host responds by computing and returning cryptographic digests over the pending Message.

Because the challenge is directed at the Sending Host using address information independently verified against the Sender's domain, a successful response demonstrates that the Sending Host is reachable at an authorised address and is genuinely in possession of the Message being sent. This strengthens both Sender validation and Message validation, and imposes a small cost on the sender that helps deter unsolicited bulk messaging.

The challenge is optional and performed at the discretion of the Receiving Host, allowing protocol assurances to be traded against efficiency. The detailed mechanism is defined by the protocol specification rather than this document.

# Scope of this Document and Request for Feedback

This document deliberately describes only the motivation, design goals and core architecture of fmsg. It does not define message encodings, the on-the-wire protocol procedures, or transport bindings. Those details exist today in an experimental specification [FMSG-SPEC] and reference implementation [FMSGD], and are intended to be specified separately as the work matures.

The anticipated document structure separates the work into:

- this architectural overview;
- a message format document defining the binary message structure;
- a host-to-host protocol document defining the exchange procedures, including the challenge-response mechanism and message acceptance; and
- one or more transport binding documents (for example, a TCP and TLS binding).

The author is seeking feedback from the IETF community on the following questions, rather than a detailed review of the underlying specification at this stage:

1. Is the IETF an appropriate venue for this work, and is the Applications and Real-Time (ART) area, by way of the DISPATCH working group, the right place to determine its disposition?
2. Is there interest in standardising a domain-level messaging protocol distinct from Internet email and from proprietary instant messaging systems?
3. Is the proposed scope appropriate, in particular the decision to standardise only host-to-host message exchange while leaving user identity, authentication, message retrieval and client behaviour out of scope?
4. Are there existing or in-progress efforts with which this work should be aligned or reconciled?

# Security Considerations

The fmsg architecture is designed around validation before acceptance. A Receiving Host validates a Message before accepting it into a Thread. This model ensures that sender verification, message integrity and recipient policy are evaluated before a Message becomes part of the recipient's conversation history.

Message acceptance comprises three validation stages: Sender validation, Message validation and Recipient validation. Together these establish that the Sending Host is authorised to send on behalf of the Sender's domain, that the Message is valid and consistent with the Thread, and that the Message satisfies recipient-specific policy.

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

