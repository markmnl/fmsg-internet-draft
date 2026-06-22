---
title: "fmsg: Structured Host-to-Host Messaging with Verifiable Threads"
category: info

docname: draft-mennell-art-fmsg-structured-messaging-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Internet Engineering Steering Group"
workgroup: "Applications and Real-Time Area"
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

...

--- abstract

fmsg defines a protocol with structured binary messages where messages can link to a previous message using a cryptographic hash, forming verifiable threads. Messages are sent via a sender's domain host to one or more recipient hosts. Receiving hosts verify message size, recipient policies and message ancestory before accepting message content. Only participants of a message can reply to that message, additional recipients can be added to a message after it has been sent.

A distinguishing feature of the fmsg protocol is a callback challenge to the sending host during message exchange, at the receiving host's discretion. Processing such a challenge requires the sending host being reachable and compute additional digests over the pending message.

This document describes the goals, principles and core architecture of fmsg. A full experimental specification including wire format, protocol steps and implementation is avaliable at [https://github.com/markmnl/fmsg](https://github.com/markmnl/fmsg). The purpose of this document is to describe the architecture and solicit IETF venue and scope feedback before the full specification is split into applicable standards-track deliverables.



--- middle

# Introduction

TODO Introduction


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
