---
title: QUIC Disable Encryption
abbrev: QUIC-DIS-ENCRYPT
docname: draft-banks-quic-disable-encryption-00
category: exp
date: 2020

stand_alone: yes

ipr: trust200902
area: Transport
kw: Internet-Draft

coding: us-ascii
pi: [toc, sortrefs, symrefs, comments]

author:
  -
    ins: N. Banks
    name: Nick Banks
    org: Microsoft Corporation
    email: nibanks@microsoft.com
	country: U.S.A

--- abstract

The disable_1rtt_encryption transport parameter can be used to negotiate the
disablement of encryption on 1-RTT packets, allowing for reduced CPU load and
improved performance.  This extension is only meant to be used in environments
where both endpoints completely trust the path between themselves; not, for
instance, on the open internet.

--- middle

# Introduction

By default QUIC Transport Protocol {{!I-D.ietf-quic-transport}} provides secured
(authenticated and encrypted) connections via a TLS handshake.  The handshake
allows for the endpoints to be authenticated by a certificate and then securely
generates shared secrets to encrypt the QUIC packet traffic.  Post-handshake,
this packet encryption can occupy a considerable percentage of CPU usage,
depending on the scenario.  Additionally, there are scenarios where the
protections given by this encryption are either unnecessary or unwanted.  For
these scenarios, this document defines an extension to the QUIC protocol to
allow for mutually participating endpoints to negotiate the disablement of
encryption for the 1-RTT packets sent after the handshake.

## Terms and Definitions

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

## Applicable Scenarios for Use

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

# Specification

The disable_1rtt_encryption transport parameter used for negotiating the use
of the extension is defined below.

## Disable 1-RTT Encryption Transport Parameter

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

## Negotiating the Extension

The payload sent in the transport parameter by the client, along with any other
information the server has about the client (such as IP address) may be used to
negotiate the extension on the server side.  The TP payload could be considered
a key or identifier used by the server to verify the client should be allowed to
disable encryption.  These additional security measures are optional, but
RECOMMENDED to ensure encryption is not accidentally enabled when it should not
be.

## Disabling 1-RTT Encryption

When the extension is negotiated, all aspects of encryption on 1-RTT packets are
removed:

 - Header protection
 - Payload protection
 - AEAD tag

This effectively gives the transport an additional 16 bytes per packet to be
used for payload, since it is no longer including an AEAD tag.

Because the AEAD tag is removed along with the encryption, the UDP checksum
must be relied upon to determine any packet corruption.

## Interactions with Path Changes

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

This document registers a new value in the QUIC Transport Parameter
Registry:

Value: TBD (using value 0xBAAD in early deployments)

Parameter Name: disable_1rtt_encryption

Specification: Indicates disabled 1-RTT encryption is being negotiated

--- back
