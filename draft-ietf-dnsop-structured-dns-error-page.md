---
title: "Structured Data for Filtered DNS"
abbrev: "Data for Filtered DNS"
category: info

docname: draft-ietf-dnsop-structured-dns-error-page-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Internet"
workgroup: "Adaptive DNS Discovery"
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: "dnsop"
  type: "Working Group"
  mail: "dnsop@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/dnsop/"
  github: "danwing/dnsop2"
  latest: "https://danwing.github.io/dnsop2/draft-ietf-add-dns-error-page.html"

author:
 -
    fullname: Dan Wing
    organization: Citrix
    email: "dwing-ietf@fuggles.com"
 -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    email: "kondtir@gmail.com"
 -
    fullname: Neil Cook
    organization: Open-Xchange
    email: "neil.cook@noware.co.uk"
 -
    fullname: Mohamed Boucadair
    organization: Orange
    email: "mohamed.boucadair@orange.com"


normative:
  RFC2119:
  RFC7493:
  RFC7159:
  RFC6838:
  RFC6891:
  RFC8174:
  RFC8310:
  RFC8126:
  RFC5198:

informative:
  IANA-MediaTypes:
     title: Media Types
     author:
       -
         organization: "IANA"
     target: https://www.iana.org/assignments/media-types
     date: false

  RPZ:
     title: Response Policy Zone
     target: https://dnsrpz.info
     date: false

  Chrome-Translate:
     title: "Change Chrome languages & translate webpages"
     target: https://support.google.com/chrome/answer/173424
     date: false

  RFC5625:
  RFC7858:
  RFC8484:
  RFC8499:
  RFC8792:
  RFC8914:
  RFC8259:
  RFC9250:


--- abstract

TODO Abstract


--- middle

# Introduction

DNS filters are deployed for a variety of reasons including endpoint
security, parental filtering, and filtering required by law
enforcement. Network-based security solutions such as firewalls and
Intrusion Prevention Systems (IPS) rely upon network traffic
inspection to implement perimeter-based security policies and operate
by filtering DNS responses. In a home, DNS filtering is used for the
same reasons as above and additionally for parental control. Internet
Service Providers typically block access to some DNS domains due to a
requirement imposed by an external entity (e.g., law enforcement
agency) also performed using DNS-based content filtering.

Users of DNS services which perform filtering may wish to receive more
information about such filtering to resolve problems with the filter
-- for example to contact the administrator to allowlist a domain that
was erroneously filtered or to understand the reason a particular
domain was filtered. With that information, the user can choose
another network, open a trouble ticket with the DNS administrator to
resolve erroneous filtering, log the information, or other uses.

For both DNS filtering mechanisms described in Section 4 of (Section
3), the DNS server can return extended error codes Blocked, Censored,
Filtered, or Forged Answer defined in Section 4 of [RFC8914]. However,
these codes only explain that filtering occurred but lack detail for
the user to diagnose erroneous filtering.

No matter which type of response is generated (forged IP address(es), NXDOMAIN or empty answer, even with an extended error code), the user who triggered the DNS query has little chance to understand which entity filtered the query, how to report a mistake in the filter, or why the entity filtered it at all. This document describes a mechanism to provide such detail.

One of the other benefits of this approach is to eliminate the need to "spoof" block pages for HTTPS resources. This is achieved since clients implementing this approach would be able to display a meaningful error message, and would not need to connect to such a block page. This approach thus avoids the need to install a local root certificate authority on those IT-managed devices.

This document describes a format for computer-parsable data in the EXTRA-TEXT field of Extended DNS Errors [RFC8914]. It updates Section 2 of Extended DNS Errors [RFC8914] which says the information in EXTRA-TEXT field is intended for human consumption (not automated parsing).

This document does not recommend DNS filtering but provides a mechanism for better transparency to explain to the users why some DNS queries are filtered.



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
