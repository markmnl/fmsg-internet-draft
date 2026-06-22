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
 - sparkling distributed ledger
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

fmsg defines structured binary messages and an exchange protocol where messages link to a previous message using a cryptographic hash, forming verifiable threads. Messages are sent via a domain host to one or more recipient hosts. Receiving hosts verify message size, recipient policies and replay status before accepting message content.

A distinguishing feature of fmsg is an automatic callback challenge to the sending host during message exchange, at the receiving host's discretion. Processing such a challenge requires being reachable and computing additional digests over the pending message, helping receivers mitigate low-effort, one-way and invalid messages before accepting the full message.

This document describes the goals, principles and core architecture of fmsg. A more complete specification including full wire format, protocol steps and implementation already exist at [https://github.com/markmnl/fmsg](https://github.com/markmnl/fmsg). The purpose of this document is to describe the architecture and solicit IETF venue and scope feedback before the full specification is split into strandards-track deliverables.



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
