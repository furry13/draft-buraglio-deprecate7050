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

normative:

informative:
  RFC7050:
  RFC8781:

--- abstract

[RFC7050] describes a method for detecting the presence of DNS64 and for learning the IPv6 prefix used for protocol translation on an access network. This methodology depends on the existence of a well-known IPv4-only fully qualified domain name "ipv4only.arpa.". Because newer methods exist that lack the requirement of a higher level protocol, instead using existing operations in the form of native router advertisements, discovery of the IPv6 prefix used for protocol translation is deprecated to legacy status.

--- middle


# Introduction

[RFC8781] describes a Neighbor Discovery option to be used in Router Advertisements (RAs) to communicate prefixes of Network Address and Protocol Translation from IPv6 clients to IPv4 servers (NAT64) to hosts. 


# Conventions and Definitions

NAT64

DNS64

Router Advertisement

pref64

{::boilerplate bcp14-tagged}

# Existing issues with RFC 7050

IPsec limitations, lack of implementation.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
