---
title: "QUIC Disable Encryption"
abbrev: "QUIC Disable Encryption"
docname: draft-banks-quic-disable-encryption
date: {DATE}
category: exp
ipr: trust200902
area: Transport
workgroup: QUIC

stand_alone: yes
pi: [toc, sortrefs, symrefs, docmapping]

author:
  -
    ins: N. Banks
    name: Nick Banks
    org: Microsoft Corporation
    email: nibanks@microsoft.com

normative:

  QUIC-TRANSPORT:
    title: "QUIC: A UDP-Based Multiplexed and Secure Transport"
    date: {DATE}
    seriesinfo:
      Internet-Draft: draft-ietf-quic-transport
    author:
      -
        ins: J. Iyengar
        name: Jana Iyengar
        org: Fastly
        role: editor
      -
        ins: M. Thomson
        name: Martin Thomson
        org: Mozilla
        role: editor

  QUIC-TLS:
    title: "Using Transport Layer Security (TLS) to Secure QUIC"
    date: {DATE}
    seriesinfo:
      Internet-Draft: draft-ietf-quic-tls-latest
    author:
      -
        ins: M. Thomson
        name: Martin Thomson
        org: Mozilla
        role: editor
      -
        ins: S. Turner
        name: Sean Turner
        org: sn3rd
        role: editor


--- abstract

This document describes a method for negotiating the disablement of encryption
on 1-RTT packets, allowing for reduced CPU load and improved performance.

--- middle

# Introduction

TODO

# Conventions and Definitions

{::boilerplate bcp14}

This document uses terms and notational conventions from {{QUIC}}.

# Disable 1-RTT Encryption Transport Parameter

The disable_1rtt_encryption transport parameter can be sent by both a client and
server.  The transport parameter is sent with an empty value; an endpoint that
understands this transport parameter MUST treat the receipt of a non-empty value
as a connection error of type TRANSPORT_PARAMETER_ERROR.

Advertising the disable_1rtt_encryption transport parameter indicates that the
endpoint wishes to disable encryption for 1-RTT packets.  Both sides must
advertise support for the feature for it to be considered successfully
negotiated.
