---
title: BRSKI enrollment for Smart Pledges 
abbrev: SmartPledge
docname: draft-richardson-anima-smartpledge-00

# stand_alone: true

ipr: trust200902
area: Internet
wg: 6tisch Working Group
kw: Internet-Draft
cat: info

coding: us-ascii
pi:    # can use array (if all yes) or hash here
  toc: yes
  sortrefs:   # defaults to yes
  symrefs: yes

author:


- ins: M. Richardson
  name: Michael Richardson
  org: Sandelman Software Works
  email: mcr+ietf@sandelman.ca

- ins: J. Latour
  name: Jacques Latour
  org: CIRA Labs
  email: Jacques.Latour@cira.ca

- ins: F. Khan
  name: Faud Khan
  org: Twelve Dot Systems
  email: faud.khan@twelvedot.com

normative:
  RFC2119:
  I-D.ietf-anima-bootstrapping-keyinfra:
  dpp:
    title: "Device Provisioning Protocol Specification"
    format:
      pdf: https://github.com/kcdtv/wpa3/blob/master/Device_Provisioning_Protocol_Specification_v1.0.pdf
    target: "https://www.wi-fi.org/downloads-registered-guest/Device_Provisioning_Protocol_Draft_Technical_Specification_Package_v0_0_23_0.zip/31255"
  iso18004:
    title: "Information technology — Automatic identification and data capture techniques — Bar code symbology — QR Codes (ISO/IEC 18004:2015)"
    target: "https://github.com/yansikeim/QR-Code/blob/master/ISO%20IEC%2018004%202015%20Standard.pdf"
informative:
  RFC4291:
  RFC7217:

--- abstract

This document details the mechanism used for initial enrollment by
a smartphone into a BRSKI based enrollment system.

There are two key differences in assumption from
{{I-D.ietf-anima-bootstrapping-keyinfra}}: that the intended registrar has
Internet, and that the Pledge has no user-interface.

The variation on BRSKI is intended to be used in the situation where the
registrar device is new out of the box and is the intended gateway to the
Internet (such as a home gateway), but has not yet been configured.  This
work is also intended as a transition to the Wi-Fi Alliance work on the
Device Provisioning Protocol.

--- middle

# Introduction

The problem of bootstrapping a new device is described at length in
{{I-D.ietf-anima-bootstrapping-keyinfra}} (BRSKI).  The problem that BRSKI
solves is the case of a smart, properly configured network with a minimum of
network connectivity (or previously pre-previoned with nonceless vouchers),
and a relatively stupid new device (the Pledge), which lacks a network
interface.

The BRSKI problem is one of trust: how does the new device trust that it has
found the correct network to join, and how does the new network become
convinced that the new device is a device that is intended to join.
BRSKI solves the problem well for the case where the network is well
connected and easily talk to the device's Manufacturer Authorized Signing
Authority (MASA), while providing appropriate proxy mechanisms to enable the
new pledge to communicate it's proximity assertion to the MASA as well.

This document is about a variation of the problem: when the new device being
introduce has no network connectivity, but a new device is intended to
serve as the Registrar for the network.  This new device is likely a home (or
small office) gateway, and until it is properly configured there will be
no direct network connectivity.

There are a number of protocols that permit an ISP to consider a new router
brought into a home to be a new pledge to the ISPs' network, and for that new
device to integrated into the ISP's (autonomic) network. BRSKI can be used
itself, and there are ways to use the Broadband Form's TR-069 to bootstrap
the device in this way.  This document is not about the situation where
the router device is intended to belong to the ISP, but about the situation
where the home user intends to own and control the device.

## Additional Motivation

The Wi-Fi Alliance has released the Device Provisioning Protocol {{dpp}}.
The specification is not public.  The specification relies on being able to
send and receive 802.11 Public Action frames, as well as Generic
Advertisement Service (GAS) Public Action frames.  Access to send new layer-2
frames is generally restricted in most smartphone operating systems (iOS,
Android). At present there are no known public APIs that a generic
application writer could use, and therefore the smart-phone side of the DPP
can only be implemented at present by the vendors of those operating systems.

As both dominant vendors have competing proprietary mechanisms, it is unclear
if generic applications will be produced soon.  It is there impossible for a
vendor an a smart-appliance to independantly produce an application that can
do proper DPP in 2018.

In addition to the above concern, DPP is primary concerned about provisioning
WiFi credentials to devices.  It assumes that the WiFi Access Point is
already provisioned and functioning correctly.

The smartpledge enrollment described in this document is about securely
initializing the administrative connection with a device that is the WiFi
Access Point. 

# Terminology          {#Terminology}

The following terminology is copied from {{I-D.ietf-anima-bootstrapping-keyinfra}}

enrollment: The process where a device presents key material to a network and acquires a
network specific identity. For example when a certificate signing request is presented to a
certification authority and a certificate is obtained in
response.

pledge: The prospective device, which has an identity installed at the
factory.

smartpledge: The prospective administrator device, usually a smartphone
equipped with a QR capable camera, wifi and 3G connectivity.

adolescent router (AR): a home router or device containing a registrar. The device does not yet
have network connectivity, and has no administrator.

# Requirements Language {#rfc2119}

In this document, the key words "MUST", "MUST NOT", "REQUIRED",
"SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY",
and "OPTIONAL" are to be interpreted as described in BCP 14, RFC 2119
{{RFC2119}} and indicate requirement levels for compliant STuPiD
implementations.

# Assumptions and Required Setup

The first assumption is that intended device owner is active and is
present. The device owner has a smart-phone that is capable of using Wi-Fi or
being wired into the adolescent router (AR).

The second assumption is that the device has a QR code printed on the outside
of the unit, and/or provided with the packaging/documentation.  The QR code
is as specified in section 5.3 of {{dpp}}, with the additions specified in {{qrextra}}




# Protocol Description

## Quick Response Code (QR code) {#qrextra}

Section 5.3 of {{dpp}} describes the contents for an {{iso18004}} image.
It specifies content that starts with DPP:, and the contains a series of
semi-colon (;) deliminated section with a single letter and colon. This
markup is terminated with a double semi-colon.

Although no amending formula is defined in DPP 1.0, this document is defining
two extensions.  This requires amending the ABNF from section 5.2.1 as
follows:

    dpp-qr = “DPP:” [channel-list “;”] [channel-list “;”]
             [mac “;”] [information “;”] public-key
             [";" llv6-addr ] [";" smartpledge ] [";" essid ] “;;”
    llv6-addr = "L:" 8*hex-octet
    essid     = "E:" *(%x20-3A / %x3C-7E) ; semicolon not allowed
    smartpledge = "S:" *(%x20-3A / %x3C-7E) ; semicolon not allowed

While the ABNF defined in the {{dpp}} document assumes a specific order
(C:, M:, I:, K:) the tags can come in any order.   However,
in order to make interoperation with future DPP-only clients
as seamless as possible, the extensions suggested here are placed at the end
of the list.  This is consistent with the Postel Principle.

### The SmartPledge Attribute

The *smartpledge* attribute indicates that the device is capable of the
protocol specified in this document, and can provide additional parameters
(TBD).

### Link-Layer Address Attribute

The *llv6-addr* attribute is optional.  When present, it specifies the IPv6
Link-Local address at which the adolescent router is listening.  If not
specified, then the link-local address may be formed according to the
historical (privacy-violating) process described in {{RFC4291}} Appendix A.
The *llv6-addr* attribute is present so that devices that have
implemented {{RFC7217}} stable addresses can express that address clearly.

### ESSID Name Attribute

The *essid* attribute provides the name of the 802.11 network to which the
*smartpledge* SHOULD join in order to reach the AR.  If this attribute is
absent, then it defaults to "BRSKI".

## Artifacts

TBD

# Security Considerations

TBD.

# IANA Considerations

TBD.

# Acknowledgements

This work was supported by the Canadian Internet Registration Authority (cira.ca).

--- back

