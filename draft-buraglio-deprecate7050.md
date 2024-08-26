---
title: "Deprication of DNS64 for Discovery of IPv6 Prefix Used for IPv6 Address Synthesis"
abbrev: "Deprication of DNS64 for Discovery of IPv6 Prefix Used for IPv6 Address Synthesis"
category: info

docname: draft-buraglio-deprecate7050-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - IPv6
 - DNS64
venue:
  group: 6man
  type: Working Group
  mail: ipv6@ietf.org
  arch: https://datatracker.ietf.org/wg/6man/about/
  github: "buraglio/draft-buraglio-deprecate7050"
  latest: "https://buraglio.github.io/draft-buraglio-deprecate7050/draft-buraglio-deprecate7050.html"

author:
 -
    fullname: Nick Buraglio
    organization: Energy Sciences Network
    email: "buraglio@forwardingplane.net"
 -
    fullname: Tommy Jensen
    organization: Microsoft
    email: "tojens.ietf@gmail.com"
normative:

informative:
  RFC6105:
  RFC7050:
  RFC8781:
  RFC9463:
  RFC9499:

--- abstract

[RFC7050] describes a method for detecting the presence of DNS64 and for learning the IPv6 prefix used for protocol translation ([RFC6145]). This methodology depends on the existence of a well-known IPv4-only fully qualified domain name "ipv4only.arpa.". Because newer methods exist that lack the requirement of a higher level protocol, instead using existing operations in the form of native router advertisements, discovery of the IPv6 prefix used for protocol translation using [RFC7050] is deprecated to legacy status.

--- middle


# Introduction

[RFC8781] describes a Neighbor Discovery option to be used in Router Advertisements (RAs) to communicate prefixes of Network Address and Protocol Translation from IPv6 clients to IPv4 servers (NAT64) to hosts. This approach has the advantage of using the same communication channel IPv6 clients use to discover other network configurations such as the network's default route. This means network administrators can secure this configuration along with other configurations IPv6 requires using a single approach such as RA Guard [RFC6105]. 


# Conventions and Definitions

NAT64

DNS64

Router Advertisement

pref64

{::boilerplate bcp14-tagged}

# Existing issues with RFC 7050

In addition to creating a wider attack surface for IPv6 deployments, [RFC7050]  has additional challenges worth noting to justify declaring it legacy.

## Definition of secure channel {secure-channel-def}

[RFC7050] requires a node's communication channel with a DNS64 server to be a "secure channel" which it defines to mean "a communication channel a node has between itself and a DNS64 server protecting DNS protocol-related messages from interception and tampering." This need is redundant when another communication mechanism of IPv6-related configuration, specific Router Advertisements, can already be defended against tampering by RA Guard [RFC6105]. Requiring nodes to implement two defense mechanisms when only one is necessary when [RFC8781] is used in place of [RFC7050] creates unnecessary risk.

## Secure channel example of IPsec

One of the two examples [RFC7050] defines to qualify a communication channel with a DNS64 server is the use of an "IPsec-based virtual private network (VPN) tunnel". As of the time of this writing, this is not supported as a practice by any common operating system DNS client. While they could, there have also since been multiple mechanisms defined for performing DNS-specific encryption such as those defined in [RFC9499] that would be more appropriately scoped to the applicable DNS traffic. These are also compatible with encrypted DNS advertisement by the network using Discovery of Network-designated Resolvers [RFC9463] that would ensure the clients know in advance that the DNS64 server supported the encryption mechanism.

## Secure channel example of link layer encryption

The other example given by [RFC7050] that would allow a communication channel with a DNS64 server to qualify as a "secure channel" is the use of a "link layer utilizing data encryption technologies". As of the time of this writing, most common link layer implementations use data encryption already with no extra effort needed on the part of network nodes. While this appears to be a trivial way to satisfy this requirement, it also renders the requirement meaningless since any node along the path can still read the higher-layer DNS traffic containing the translation prefix. This seems to be at odds with the definition of "secure channel" as explained in {{secure-channel-def}}.

# Preferred approach

The prefix discovery mechanism defined in [RFC8781] is preferred to use over [RFC7050].

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
