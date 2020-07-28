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
on 1-RTT packets, allowing for reduced CPU load and improved performance.  This
extension is only meant to be used in environments where both endpoints
completely trust the path between themselves; not, for instance, on the open
internet.

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

 - Trusted Environment/Path - There are scenarios or environments where there is
   no need for the additional security measures of QUIC encryption; such as
   walled-gardens or tunneled connections.  These scenarios are either already
   trusted or secured by other means.

 - Performance Testing - When the actual contents of the QUIC packets are
   unimportant and the goal is purely to measure the performance characteristics
   of either the network, machine or QUIC implementation without encryption.

# Disable 1-RTT Encryption Transport Parameter

The disable_1rtt_encryption transport parameter can be sent by both a client and
server.  The transport parameter is sent with an optional variable-length value
by the client and an empty value by the server; a client that understands this
transport parameter MUST treat the receipt of a non-empty value as a connection
error of type TRANSPORT_PARAMETER_ERROR.

Advertising the disable_1rtt_encryption transport parameter indicates that the
endpoint wishes to disable encryption for 1-RTT packets.  Both sides must
advertise support for the feature for it to be considered successfully
negotiated.

If successfully negotiated, all packets that would normally be encrypted with
the 1-RTT key are instead sent as cleartext; both header and packet protections
are disabled.

# Negotiating the Extension

The payload sent in the transport parameter by the client, along with any other
information the server has about the client (such as IP address) may be used to
negotiate the extension on the server side.  The TP payload could be considered
a key or identifier used by the server to verify the client should be allowed to
disable encryption.  These additional security measures are optional, but
RECOMMENDED to ensure encryption is not accidentally enabled when it should not
be.

# Disabling 1-RTT Encryption

When the extension is negotiated, all aspects of encryption on 1-RTT packets are
removed:

 - Header protection
 - Payload protection
 - AEAD tag

This effectively gives the transport an additional 16 bytes per packet to be
used for payload, since it is no longer including an AEAD tag.

Because the AEAD tag is removed along with the encryption, the UDP checksum
must be relied upon to determine any packet corruption.

# Interactions with Path Changes

When making the trust determination about the path, each endpoints must take
into account possible path changes; NAT rebinding for instance.  An endpoint
MUST NOT enable enable this extension if it is possible for the path to change
during the connection to some untrusted state.

Additionally, a client MUST NOT try to migrate to any path that is untrusted
if this extension is negotiated.  If a server receives a packet for a connection
with this extension negotiated on an untrusted path, it MUST silently drop the
packet.

# Security Considerations

Disabling encryption for 1-RTT packets has some fairly obvious security
drawbacks:

 - Packets can be read, modified and injected by any middleboxes

This extension is not meant to be used for any practical application protocol on
the open internet.  Internet facing servers MUST NOT enable this extension.
Clients that do not trust their network and path to the server MUST NOT enable
this extension.

This extension does not modify the packet protections used during the handshake,
so the handshake can still be securely authenticated.  This prevents scenarios
where one endpoint might trust (or think it trusts) the path, but the other
endpoint does not, and a man-in-the-middle tries to force this extension to be
used.

To prevent accidental use of the feature on production systems it is
RECOMMENDED for servers to have additional measures such as IP filtering or a
security key.

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
