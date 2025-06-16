---
title: "Prefer use of RFC8781 for Discovery of IPv6 Prefix Used for IPv6 Address Synthesis"
abbrev: "Prefer RFC8781"
category: info

docname: draft-ietf-v6ops-prefer8781-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ops
workgroup: v6ops
keyword:
 - IPv6
 - DNS64
 - PREF64
 - NAT64
 - CLAT
venue:
  group: v6ops
  type: Working Group
  mail: dnsop@ietf.org
  arch: https://datatracker.ietf.org/wg/v6ops/about/
  github: "buraglio/draft-nbtjjl-dnsop-prefer8781"
  latest: "https://github.com/buraglio/draft-nbtjjl-v6ops-prefer8781"

author:
 -
    fullname: Nick Buraglio
    organization: Energy Sciences Network
    email: "buraglio@forwardingplane.net"
 -
    fullname: Tommy Jensen
    organization: Microsoft
    email: "tojens.ietf@gmail.com"
 -
    fullname: Jen Linkova
    organization: Google
    email: "furry13@gmail.com"
normative:
  RFC7050:
  RFC8781:

informative:
  RFC4861:
  RFC4862:
  RFC6105:
  RFC6144:
  RFC6145:
  RFC6146:
  RFC7915:
  RFC6147:
  RFC6877:
  RFC8880:
  RFC9463:
  RFC9499:

--- abstract

On networks providing IPv4-IPv6 translation (NAT64, RFC7915), hosts and other endpoints might need to know the IPv6 prefix used for translation (the NAT64 prefix).
While RFC7050 defined a DNS64-based prefix discovery mechanism, more robust methods have since emerged.
This document provides updated guidelines for NAT64 prefix discovery, deprecating the RFC7050 approach in favor of modern alternatives (e.g., RFC8781) whenever available.

--- middle

# Introduction

NAT64 devices translating between IPv4 and IPv6 packet headers ([RFC7915]) employ a NAT64 prefix to map IPv4 addresses into the IPv6 address space, and vice versa.
When a network provides NAT64 services, it is advantageous for hosts and endpoints to acquire the network's NAT64 prefix (PREF64).
Discovering the PREF64 enables endpoints to:

  * implement the customer-side translator (CLAT) functions of the 464XLAT architecture [RFC6877];
  * translate the IPv4 literal to an IPv6 literal (Section 7.1 of [RFC8305]);
  * perform local DNS64 ([RFC6147]) functions.

Dynamic PREF64 discovery is often essential, particularly for unmanaged or mobile endpoints, where static configuration is impractical.
While [RFC7050] introduced the first DNS64-based mechanism for PREF64 discovery, subsequent methods have been developed to address its limitations.


For instance, [RFC8781] defines a Neighbor Discovery ([RFC4861]) option for Router Advertisements (RAs) to convey PREF64 information to hosts.
This approach offers several advantages (Section 3 of [RFC8781]), including fate sharing  with other host network configuration parameters.

Due to fundamental shortcomings of the [RFC7050] mechanism ({{issues}}), [RFC8781] is the preferred solution for new deployments.
Implementations should strive for consistent PREF64 acquisition methods.
The DNS64-based mechanism of [RFC7050] should be employed only when RA-based PREF64 delivery is unavailable, or as a fallback for legacy systems incapable of processing the PREF64 RA option.

# Conventions and Definitions

CLAT: A customer-side translator (XLAT) that complies with [RFC6145].

DNS64: a mechanism for synthesizing AAAA records from A records, defined in [RFC6147].

NAT64: a mechanism for translating IPv6 packets to IPv4 packets and vice versa.  The translation is done by translating the packet headers according to the IP/ICMP Translation Algorithm defined in [RFC7915]. NAT64 translators can operate in stateless or stateful mode ([RFC6144]).

PREF64 (or NAT64 prefix): An IPv6 prefix used for IPv6 address synthesis and for network addresses and protocols translation from IPv6 clients to IPv4 servers, [RFC6146].

Router Advertisement (RA): A packet used by Neighbor Discovery [RFC4861] and SLAAC to advertise the presence of the routers, together with other IPv6 configuration information.

SLAAC:  StateLess Address AutoConfiguration, [RFC4862]

{::boilerplate bcp14-tagged}

# Existing issues with RFC 7050 {#issues}

DNS-based method of discovering the NAT64 prefix introduces some challenges, which make this approach less preferable than most recently developed alternatives (such as PREF64 RA option, [RFC8781]).
This section outlines the key issues, associated with [RFC7050].

## Dependency on Network-Provided Recursive Resolvers

Fundamentally, the presence of the NAT64 and the exact value of the prefix used for the translation are network-specific attributes.
Therefore, [RFC7050] requires the device to use the DNS64 resolvers provided by the network.
If the device or an application is configured to use other recursive resolvers or runs a local recursive resolver, the corresponding name resolution APIs and libraries are required to recognize 'ipv4only.arpa.' as a special name and give it special treatment.
This issue and remediation approach are discussed in [RFC8880].
However, it has been observed that very few [RFC7050] implementations support [RFC8880] requirements for special treatment of 'ipv4only.arpa.'.
As a result, configuring such systems and applications to use resolvers other than the one provided by the network breaks the PREF64 discovery, leading to degraded user experience.

VPN clients often override the host's DNS configuration, for example, by configuring enterprise DNS servers as the host's recursive resolvers and forcing all name resolution through the VPN.
These enterprise DNS servers typically lack DNS64 functionality and therefore cannot provide information about the PREF64 used within the local network.
Consequently, this prevents the host from discovering the necessary PREF64, negatively impacting its connectivity on IPv6-only networks

## Network Stack Initialization Delay

When using SLAAC, an IPv6 host typically requires a single RA to acquire its network configuration.
For IPv6-only hosts, timely PREF64 discovery is critical, particularly for those performing local DNS64 or NAT64 functions, such as CLAT.
Until the PREF64 is obtained, the host's IPv4-only applications and communication to IPv4-only destinations are impaired.
The mechanism defined in [RFC7050] does not bundle PREF64 information with other network configuration parameters.

## Latency in Updates Propagation

Section 3 of [RFC7050] requires that the node SHALL cache the replies received during the PREF64 discovery and SHOULD repeat the discovery process ten seconds before the TTL of the Well-Known Name's synthetic AAAA resource record expires.
As a result, once the PREF64 is discovered, it will be used until the TTL expired, or until the node disconnects from the network.
There is no mechanism for an operator to force the PREF64 rediscovery on the node without disconnecting the node from the network.
If the operator needs to change the PREF64 value used in the network, they need to proactively reduce the TTL value returned by the DNS64 server.
This method has two significant drawbacks:

*  Many networks utilize external DNS64 servers and therefore have no control over the TTL value.
*  The PREF64 changes need to be planned and executed at least TTL seconds in advance. If the operator needs to notify nodes that a particular prefix must not be used (e.g. during a network outage or if the nodes learnt a rogue PREF64 as a result of an attack), it might not be possible without interrupting the network connectivity for the affected nodes.

## Multihoming Implications

According to Section 3 of [RFC7050], a node MUST examine all received AAAA resource records to discover one or more PREF64s and MUST utilize all learned prefixes.
However, this approach presents challenges in some multihomed topologies where different DNS64 servers belonging to different ISPs might return different PREF64s.
In such cases, it is crucial that traffic destined for synthesized addresses is routed to the correct NAT64 device and the source address selected for those flows belongs to the prefix from that ISP's address space.
In other words, the node needs to associate the discovered PREF64 with upstream information, including the IPv6 prefix and default gateway.
Currently, there is no reliable way for a node to map a DNS64 response (and the prefix learned from it) to a specific upstream in a multihoming scenario.
Consequently, the node might inadvertently select an incorrect source address for a given PREF64 and/or send traffic to the incorrect uplink.

## Security Implications

As discussed in Section 7 of [RFC7050], the DNS-based PREF64 discovery is prone to DNS spoofing attacks.
In addition to creating a wider attack surface for IPv6 deployments, [RFC7050] has other security challenges worth noting to justify declaring it legacy.

### Definition of secure channel {#secure-channel-def}

[RFC7050] requires a node's communication channel with a DNS64 server to be a "secure channel" which it defines to mean "a communication channel a node has between itself and a DNS64 server protecting DNS protocol-related messages from interception and tampering." This need is redundant when another communication mechanism of IPv6-related configuration, specifically Router Advertisements, can already be defended against tampering by RA Guard [RFC6105]. Requiring nodes to implement two defense mechanisms when only one is necessary when [RFC8781] is used in place of [RFC7050] creates unnecessary risk.

### Secure channel example of IPsec

One of the two examples [RFC7050] defines to qualify a communication channel with a DNS64 server is the use of an "IPsec-based virtual private network (VPN) tunnel". As of the time of this writing, this is not supported as a practice by any common operating system DNS client. While they could, there have also since been multiple mechanisms defined for performing DNS-specific encryption such as those defined in [RFC9499] that would be more appropriately scoped to the applicable DNS traffic. These are also compatible with encrypted DNS advertisement by the network using Discovery of Network-designated Resolvers [RFC9463] that would ensure the clients know in advance that the DNS64 server supported the encryption mechanism.

### Secure channel example of link layer encryption

The other example given by [RFC7050] that would allow a communication channel with a DNS64 server to qualify as a "secure channel" is the use of a "link layer utilizing data encryption technologies". As of the time of this writing, most common link layer implementations use data encryption already with no extra effort needed on the part of network nodes. While this appears to be a trivial way to satisfy this requirement, it also renders the requirement meaningless since any node along the path can still read the higher-layer DNS traffic containing the translation prefix. This seems to be at odds with the definition of "secure channel" as explained in {{Section 2.2 of RFC7050}}.

# Recommendations for PREF64 Discovery

## Deployment Recommendations

Operators deploying NAT64 networks SHOULD provide PREF64 information in Router Advertisements as per [RFC8781].

### Mobile network considerations

Use of [RFC8781] may not be currently practical for networks that have more complex network control signaling or rely on slower network component upgrade cycles, such as mobile networks. These environments are encouraged to incorporate [RFC8781] when made practical by infrastructure upgrades or software stack feature additions.

## Clients Implementation Recommendations

Clients SHOULD obtain PREF64 information from Router Advertisements as per [RFC8781] instead of using [RFC7050] method.
In the absence of the PREF64 information in RAs, a client MAY choose to fall back to RFC7050.

# Security Considerations

Obtaining PREF64 information from Router Advertisements improves the overall security of an IPv6-only client as it mitigates all attack vectors related to spoofed or rogue DNS response, as discussed in Section 7 of [RFC7050].
Security considerations related to obtaining PREF64 information from RAs are discussed in Section 7 of [RFC8781].

# IANA Considerations

It is expected that there will be a long tail of both clients and networks still relying on [RFC7050] as a sole mechanism to discover PREF64 information.
Therefore IANA still need to maintain "ipv4only.arpa." as described in [RFC7050] and this document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

The authors would like to than the following people for their valuable contributions: Lorenzo Colitti, Tom Costello, Charles Eckel, Nick Heatley, Gabor Lencse and Peter Schmitt.
