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
{{I-D.ietf-anima-bootstrapping-keyinfra}} (aka BRSKI).  The problem that BRSKI
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

enrollment:
: The process where a device presents key material to a network and acquires a
network specific identity. For example when a certificate signing request is presented to a
certification authority and a certificate is obtained in
response.

pledge:
: The prospective device, which has an identity installed at the factory.

smartpledge:
: The prospective administrator device, usually a smartphone equipped with a
QR capable camera, wifi and 3G connectivity. 

adolescent router (AR):
: a home router or device containing a registrar. The device does not yet
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

The smartpledge application generates a self-signed certificate with
public/private keypair that it knows.  It may generate a unique certificate
for each manufacturer.  This certificate is called the SelfDevID.

The second assumption is that the device has a QR code printed on the outside
of the unit, and/or provided with the packaging/documentation.  The QR code
is as specified in section 5.3 of {{dpp}}, with the additions specified in {{qrextra}}

The third assumption is that the AR, at manufacturing time, has the anchor
for it's MASA (same assumption as for BRSKI pledge's).  In addition, like
the BRSKI pledge, the AR has an IDevID certificate (and associated private
key) signed by the manufacturer.

The fourth assumption is that the key in the "K:" attribute {{qrextra}} is a
different public key pair.  It MUST be different from the key used in the
IDevID.  This key is called the DPP-Keypair.

# Protocol Overview

## Scan the QR code

The operator of the smartphone invokes the smartpledge application, and scans
the QR code on the AR.  The smartpledge learns the ESSID, Public-Key,
mac-address, smartpledge URL, and link-local address of the AR.

## Enroll with the manufacturer

The smartpledge uses it's 3G, or other WiFi internet access to connect to the
manufacturer with TLS.  The smartpledge does an HTTP POST to the provided URL
using it's generated certificate as it's ClientCertificate.
As described in {{smartpledgeenroll}}, the manufacturer MAY respond with a
302 result code, and have the end user go through a web browser based process
to enroll.  After that process, a redirection will occur using OAUTH2.

The result should finally be a 201 result code, and at that URL is a new
certificate signed by the manufacturer.

## Connect to BRSKI join network

The application then reconnects the Wi-Fi interface of the smartphone to the
ESSID of the AR.   This involves normal 802.11 station attachment.  The ESSID
explicitely has no WPA or other security required on it.

There will be no DHCPv4.  A IPv6 Router Solicitation may elicit an answer
(confirming the device is there), but it is acceptable for there to be no
prefix information.  An IPv6 Neighbour Discovery is done for the IPv6
Link-Local address of the AR.  Receipt of an answer confirms that the ESSID
is correct and present.

(XXX -- not using GRASP here. Could use GRASP, but QR code is better)

## Connect to Adolescent Registrar (AR)

The smartpledge application then makes a direct (no proxy) TLS connection to
port 443 of the AR, on the IPv6 Link-Local address given.  This is as in
section 5.1 of {{I-D.ietf-anima-bootstrapping-keyinfra}}.   The smartpledge uses it's SelfDevID as the
TLS ClientCertificate, as the smartpledge does not have a manufacturer signed
IDevID.

Additionally, the AR will use it's IDevID certificate as the
ServerCertificate of the TLS conncetion.  As with other BRSKI IDevID,
it will have a MASA URL extension, as described in {{I-D.ietf-anima-bootstrapping-keyinfra}} section 2.3.2.

## Pledge Requests Voucher-Request from the Adolescent Registrar

The smartpledge generates a random nonce "SPnonce".  To this is adds ???, to
create a voucher-request challenge.
It uses the public-key of the AR, and encrypts the challenge with it.

The result is POST'ed to the new BRSKI endpoint:

    /.well-known/est/requestvoucherrequest

(format of this to be determined. SHOULD include SHA256 of public key)

## AR processing of voucher-request, request.

The AR processes this POST.  First it uses the private key that is associated
with it's QR printed public key to decrypt the voucher-request challenge.
Included in this challenge is a nonce, and also the link-local address of the
smartpledge.

The AR then forms a voucher-request identically to as described in section
5.2 of {{I-D.ietf-anima-bootstrapping-keyinfra}}.  In additoon to a randomly generated nonce for the voucher,
it includes a new header, "voucher-challenge-nonce", which is the base64 encoded
random SPnonce that the smartpledge sent.

This voucher-request is then *returned* in the POST operation to the
smartpledge.

## Smart-Pledge connects to MASA

The smartpledge application then examines the MASA URL provided in the TLS
ServerCertificate of the AR.  The smartpledge application then 


# Protocol Details

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
protocol specified in this document.  The contents of the smartpledge
attribute contains part of a URL which identifies the manufacturer of
this device, along with a unique token enabling service access to the
device. 

Short URLs are essential to fit into typical QR code space.

The smartpledge application prepends the text "https://" to the value
provided to form the full address of the smartpledge enrollment.

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

# Smart Pledge enrollment with manufacturer {#smartpledgeenroll}

TBD.

# Security Considerations

TBD.

# IANA Considerations

TBD.

# Acknowledgements

This work was supported by the Canadian Internet Registration Authority (cira.ca).

--- back

