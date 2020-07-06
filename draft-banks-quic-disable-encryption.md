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

By default all QUIC connections are authenticated and secured, via a TLS
handshake.  The handshake allows for the endpoints to be authenticated by a
certificate and then securely generates shared secrets to encrypt the QUIC
packet traffic.  Post-handshake, this packet encryption can occupy a
considerable percentage of CPU usage, depending on the scenario.  Additionally,
there are scenarios where the protections given by this encryption are either
unnecessary or unwanted.  For these scenarios, this document defines an
extension to the QUIC protocol to allow for mutually participating endpoints to
negotiate the disablement of encryption for the 1-RTT packets sent after the
handshake.

# Conventions and Definitions

{::boilerplate bcp14}

This document uses terms and notational conventions from {{QUIC}}.

# Applicable Scenarios for Use

QUIC connections are generally meant to always be encrypted, to prevent
unauthenticated middleboxes from reading or modifying the QUIC packets.  This is
the desired behavior for most environments; especially any that go over the open
internet.  There are two possible scenarios where disabling packet encryption
makes sense:

 - Performance Testing - When the actual contents of the QUIC packets are
   unimportant and the goal is purely to measure the performance characteristics
   of either the network, machine or QUIC implementation without encryption.

 - Trusted Environment/Path - There are scenarios or environments where there is
   no need for the additional security measures of QUIC encryption; such as
   walled-gardens or tunneled connections.  These scenarios are either already
   trusted or secured by other means.

# Disable 1-RTT Encryption Transport Parameter

The disable_1rtt_encryption transport parameter can be sent by both a client and
server.  The transport parameter is sent with an empty value; an endpoint that
understands this transport parameter MUST treat the receipt of a non-empty value
as a connection error of type TRANSPORT_PARAMETER_ERROR.

Advertising the disable_1rtt_encryption transport parameter indicates that the
endpoint wishes to disable encryption for 1-RTT packets.  Both sides must
advertise support for the feature for it to be considered successfully
negotiated.

If successfully negotiated, all packets that would normally be encrypted with
the 1-RTT key are instead sent as cleartext; both header and packet protections
are disabled.

# Disabling 1-RTT Encryption

When the extension is negotiated, all aspects of encryption on 1-RTT packets are
removed:

 - Header protection
 - Payload protection
 - AEAD tag

This effectively gives the transport an additional 16 bytes per packet to be
used for payload, since it is no longer including an AEAD tag.

# Security Considerations

Disabling encryption for 1-RTT packets has some fairly obvious security
drawbacks:

 - Packets can be read, modified and injected by any middleboxes

This extension is not meant to be used for any practical application protocol on
the open internet.  Internet facing servers MUST NOT enable this extension.
Clients that do not trust their network and path to the server MUST NOT enable
this extension.

Because the AEAD tag is removed along with the encryption, the UDP checksum
must be relied upon to determine any packet corruption.

# IANA Considerations

This document registers the disable_1rtt_encryption transport parameter in the
"QUIC Transport Parameters" registry established in Section 22.2 of {{QUIC}}.
The following fields are registered:

Value:
: 0xBAAD

Parameter Name:
: disable_1rtt_encryption

Status:
: Permanent

Specification:
: This document.

Date:
: Date of registration.

Contact:
: IETF QUIC Working Group (quic@ietf.org)

Notes:
: (none)

--- back

# Acknowledgments
{:numbered="false"}

TODO
