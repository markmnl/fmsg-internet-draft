---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "fmsg: Structured Host-to-Host Messaging with Verifiable Threads"
category: info

docname: draft-mennell-art-fmsg-structured-messaging
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: AREA
workgroup: WG Working Groupre
keyword:
 - messaging
 - protocol
 - host-to-host
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: USER/REPO
  latest: https://example.com/LATEST

author:
 -
    fullname: Mark Mennell
    organization: Independent
    email: markmnl@fmsg.io

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
