---
title: "Structured Error Data for Filtered DNS"
abbrev: "Structured DNS Error"
category: std
updates: 8914

docname: draft-ietf-dnsop-structured-dns-error-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Internet"
workgroup: "Adaptive DNS Discovery"
keyword:
 - Customer experience
 - Informed decision
 - transparency
 - enriched feedback
venue:
  group: "dnsop"
  type: "Working Group"
  mail: "dnsop@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/dnsop/"
  github: "ietf-wg-dnsop/draft-ietf-dnsop-structured-dns-error"
  latest: "https://ietf-wg-dnsop.github.io/draft-ietf-dnsop-structured-dns-error/draft-ietf-dnsop-structured-dns-error.html"

stand_alone: yes
pi: [toc, sortrefs, symrefs, strict, comments, docmapping]

author:
 -
    fullname: Dan Wing
    organization: Citrix Systems, Inc.
    abbrev: Citrix
    country: United States of America
    email: "dwing-ietf@fuggles.com"
 -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    city: Bangalore
    region: Karnataka
    country: India
    email: "kondtir@gmail.com"
 -
    fullname: Neil Cook
    organization: Open-Xchange
    country: United Kingdom
    email: "neil.cook@noware.co.uk"
 -
    fullname: Mohamed Boucadair
    organization: Orange
    street: Rennes
    code: 35000
    country: France
    email: "mohamed.boucadair@orange.com"


normative:

informative:
  IANA-MediaTypes:
     title: Media Types
     author:
       -
         organization: "IANA"
     target: https://www.iana.org/assignments/media-types
     date: false

  IANA-DNS:
     title: Domain Name System (DNS) Parameters, Extended DNS Error Codes
     author:
       -
         organization: "IANA"
     target: https://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml#extended-dns-error-codes
     date: false

  RPZ:
     title: Response Policy Zone
     target: https://dnsrpz.info
     date: false

  Chrome-Translate:
     title: "Change Chrome languages & translate webpages"
     target: https://support.google.com/chrome/answer/173424
     date: false

--- abstract

DNS filtering is widely deployed for network security, but filtered
DNS responses lack information for the end user to understand the
reason for the filtering. Existing mechanisms to provide detail to end
users cause harm especially if the blocked DNS response is to an HTTPS
website.

This document updates RFC 8914 by structuring the EXTRA-TEXT field of
the Extended DNS Error to provide details on the DNS filtering. Such
details can be parsed by the client and displayed, logged, or used for
other purposes. Other than that, this document does not change any
thing written in RFC 8914.

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

For the DNS filtering mechanisms described in {{techniques}} the DNS
server can return extended error codes Blocked, Censored, Filtered, or
Forged Answer defined in Section 4 of {{!RFC8914}}. However, these codes
only explain that filtering occurred but lack detail for the user to
diagnose erroneous filtering.

No matter which type of response is generated (forged IP address(es),
NXDOMAIN or empty answer, even with an extended error code), the user
who triggered the DNS query has little chance to understand which
entity filtered the query, how to report a mistake in the filter, or
why the entity filtered it at all. This document describes a mechanism
to provide such detail.

One of the other benefits of this approach is to eliminate the need to
"spoof" block pages for HTTPS resources. This is achieved since
clients implementing this approach would be able to display a
meaningful error message, and would not need to connect to such a
block page. This approach thus avoids the need to install a local root
certificate authority on those IT-managed devices.

This document describes a format for computer-parsable data in the
EXTRA-TEXT field of {{!RFC8914}}. It updates Section 2 of {{!RFC8914}} which
says the information in EXTRA-TEXT field is intended for human
consumption (not automated parsing).

This document does not recommend DNS filtering but provides a
mechanism for better transparency to explain to the users why some DNS
queries are filtered.



# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document uses terms defined in DNS Terminology {{?RFC8499}}.

"Requestor" refers to the side that sends a request. "Responder"
refers to an authoritative, recursive resolver or other DNS component
that responds to questions. Other terminology is used here as defined
in the RFCs cited by this document.

"Encrypted DNS" refers to any encrypted scheme to convey DNS messages,
for example, DNS-over-HTTPS {{?RFC8484}}, DNS-over-TLS {{?RFC7858}}, or
DNS-over-QUIC {{?RFC9250}}.


# DNS Filtering Techniques and Their Limitations {#techniques}

DNS responses can be filtered by sending a bogus (also called,
"forged") A or AAAA response, NXDOMAIN error or empty answer, or an
extended DNS error (EDE) code defined in {{!RFC8914}}. Each of these
methods have advantages and disadvantages that are discussed below:

1. The DNS response is forged to provide a list of IP addresses that
points to an HTTP(S) server alerting the end user about the reason for
blocking access to the requested domain (e.g., malware). When an
HTTP(S) enabled domain name is blocked, the network security device
(e.g., CPE, firewall) presents a block page instead of the HTTP
response from the content provider hosting that domain. If an HTTP
enabled domain name is blocked, the network security device intercepts
the HTTP request and returns a block page over HTTP. If an HTTPS
enabled domain is blocked, the block page is also served over
HTTPS. In order to return a block page over HTTPS, man in the middle
(MITM) is enabled on endpoints by generating a local root certificate
and an accompanying (local) public/private key pair. The local root
certificate is installed on the endpoint while the network security
device(s) stores a copy of the private key. During the TLS handshake,
the network security device modifies the certificate provided by the
server and (re)signs it using the private key from the local root
certificate.

   * However, configuring the local root certificate on endpoints is
     not a viable option in several deployments like home networks,
     schools, Small Office/Home Office (SOHO), and Small/ Medium
     Enterprise (SME). In these cases, the typical behavior is that
     the filtered DNS response points to a server that will display
     the block page. If the client is using HTTPS (via web browser or
     another application) this results in a certificate validation
     error which gives no information to the end-user about the reason
     for the DNS filtering. Browsers will display errors such as "The
     security certificate presented by this website was not issued by
     a trusted certificate authority" (Internet Explorer/Edge"), "The
     site's security certificate is not trusted" (Chrome), "This
     Connection is Untrusted" (Firefox), "Safari can't verify the
     identity of the website..." (Safari on MacOS). Applications might
     display even more cryptic error messages.

   * Enterprise networks do not assume that all the connected devices
     are managed by the IT team or Mobile Device Management (MDM)
     devices, especially in the quite common Bring Your Own Device
     (BYOD) scenario. In addition, the local root certificate cannot
     be installed on IoT devices without a device management tool.

   * An end user does not know why the connection was prevented and,
     consequently, may repeatedly try to reach the domain but with no
     success. Frustrated, the end user may switch to an alternate
     network that offers no DNS filtering against malware and
     phishing, potentially compromising both security and
     privacy. Furthermore, certificate errors train users to click
     through certificate errors, which is a bad security practice. To
     eliminate the need for an end user to click through certificate
     errors, an end user may manually install a local root certificate
     on a host device. Doing so, however, is also a bad security
     practice as it creates a security vulnerability that may be
     exploited by a MITM attack. When a manually installed local root
     certificate expires, the user has to (again) manually install the
     new local root certificate.

2. The DNS response is forged to provide a NXDOMAIN response to cause
the DNS lookup to terminate in failure. In this case, an end user does
not know why the domain cannot be reached and may repeatedly try to
reach the domain but with no success. Frustrated, the end user may use
insecure connections to reach the domain, potentially compromising
both security and privacy.

3. The extended error codes Blocked, Censored, and Filtered defined in
Section 4 of {{!RFC8914}} can be returned by a DNS server to provide
additional information about the cause of an DNS error. If the
extended error code "Forged Answer" defined in Section 4.5 of
{{!RFC8914}} is returned by the DNS server, the client can identify the
DNS response is forged together with the reason for HTTPS certificate
error.

4. These extended error codes do not suffer from the limitations
discussed in bullets (1) and (2), but the user still does not know the
exact reason nor he/she is aware of the exact entity blocking the
access to the domain. For example, a DNS server may block access to a
domain based on the content category such as "Malware" to protect the
endpoint from malicious software, "Phishing" to prevent the user from
revealing sensitive information to the attacker, etc. A user needs to
know the contact details of the IT/InfoSec team to raise a complaint.

5. When a resolver or forwarder forwards the received EDE option, the
EXTRA-TEXT field only conveys the source of the error (Section 3 of
{{!RFC8914}}) and does not provide additional textual information about
the cause of the error.


# I-JSON in EXTRA-TEXT field

Servers that are compliant with this specification send I-JSON data in
the EXTRA-TEXT field {{!RFC8914}} using the Internet JSON (I-JSON)
message format {{!RFC7493}}.

> Note that {{!RFC7493}} was based on {{!RFC7159}}, but {{!RFC7159}} was replaced by {{?RFC8259}}.

This document defines the following JSON names:

c: (contact)
: The contact details of the IT/InfoSec team to report mis-classified
DNS filtering. This field is structured as an array of contact URIs
(e.g., tel, sips, https). At least one contact URI MUST be
included. This field is mandatory.

j: (justification)
: UTF-8-encoded {{!RFC5198}} textual justification for this particular
DNS filtering. The field should be treated only as diagnostic
information for IT staff. This field is mandatory.

s: (suberror)
: the suberror code for this particular DNS filtering. This field is optional.

o: (organization)
: UTF-8-encoded human-friendly name of the organization that filtered this particular DNS query. This field is optional.


New JSON names MUST be defined in the IANA
"application/json+structured-dns-error" registry ({{IANA}}), and MUST
consist only of lower-case ASCII characters, digits, and hyphens (that
is, Unicode characters U+0061 through 007A, U+0030 through U+0039, and
U+002D). These names MUST be 63 characters or shorter and it is
RECOMMENDED they be as short as possible.

The text in the "j" and "o" names can include international
characters. If the text is displayed in a language not known to the
end user, browser extensions to translate to user's native language
can be used. For example, "Google Translate" extension
{{Chrome-Translate}} provided by Google on Chrome can be used to
translate the text.

To reduce packet overhead the generated JSON SHOULD be as short as
possible: short domain names, concise text in the values for the "j"
and "o" names, and minified JSON (that is, without spaces or line
breaks between JSON elements).

The JSON data can be parsed to display to the user, logged, or
otherwise used to assist the end-user or IT staff with troubleshooting
and diagnosing the cause of the DNS filtering.


# Protocol Operation

## Client Generating Request

When generating a DNS query, the client includes the Extended DNS
Error option Section 2 of {{!RFC8914}} in the OPT pseudo-RR {{!RFC6891}} to
elicit the Extended DNS Error option in the DNS response.

## Server Generating Response

When the DNS server filters its DNS response to an A or AAAA record
query, the DNS response MAY contain an empty answer, NXDOMAIN, or a
forged A or AAAA response, as desired by the DNS server. In addition,
if the query contained the OPT pseudo-RR the DNS server MAY return
more detail in the EXTRA-TEXT field as described in {{client-processing}}.

Servers may decide to return small TTL values in filtered DNS
responses (e.g., 2 seconds) to handle domain category and reputation
updates.

## Client Processing Response {#client-processing}

On receipt of a DNS response with an Extended DNS Error option, the
following actions are performed if the EXTRA-TEXT field contains valid
JSON:

* The response MUST be received over an encrypted DNS channel. If not,
  the requestor MUST discard data in the EXTRA-TEXT field.

* The response MUST be received from a DNS server which advertised EDE
  support via a trusted channel, e.g., RESINFO
  {{?I-D.reddy-add-resolver-info}}.

* Servers which don't support this specification might use plain text
  in the EXTRA-TEXT field so that requestors SHOULD properly handle
  both plaintext and JSON text in the EXTRA-TEXT field.

* The DNS response MUST also contain an extended error code of
  "Censored", "Blocked", "Filtered" or "Forged" {{!RFC8914}}, otherwise
  the EXTRA-TEXT field is discarded.

* If either of the mandatory JSON names "c" and "j" are missing or
  have empty values in the EXTRA-TEXT field, the entire JSON is
  discarded.

* The JSON name "s" MUST NOT be present with the extended error code
  "Censored".

* If a DNS client has enabled opportunistic privacy profile (Section 5
  of {{!RFC8310}}) for DoT, the DNS client will either fallback to an
  encrypted connection without authenticating the DNS server provided
  by the local network or fallback to clear text DNS, and cannot
  exchange encrypted DNS messages. Both of these fallback mechanisms
  adversely impacts security and privacy. If the DNS client has
  enabled opportunistic privacy profile for DoT, the DNS client MUST
  ignore the EXTRA-TEXT field of the EDE responses, but SHOULD process
  other parts of the response.

* If a DNS client has enabled strict privacy profile (Section 5 of
  {{!RFC8310}}) for DoT, the DNS client requires an encrypted connection
  and successful authentication of the DNS server; this mitigates both
  passive eavesdropping and client redirection (at the expense of
  providing no DNS service if an encrypted, authenticated connection
  is not available). If the DNS client has enabled strict privacy
  profile for DoT, the client MAY process the EXTRA-TEXT field of the
  DNS response. Note that the strict and opportunistic privacy
  profiles as defined in {{!RFC8310}} only apply to DoT; there has been
  no such distinction made for DoH.

* If the DNS client determines that the encrypted DNS server does not
  offer DNS filtering service, it MUST discard the EXTRA-TEXT field of
  the EDE response. For example, the DNS client can learn whether the
  encrypted DNS resolver performs DNS-based content filtering or not
  by retrieving resolver information using the method defined in
  {{?I-D.reddy-add-resolver-info}}.

* When a forwarder receives an EDE option, whether or not (and how) to
  pass along JSON information in the EXTRA-TEXT on to their client is
  implementation dependent {{?RFC5625}}. Implementations MAY choose to
  not forward the JSON information, or they MAY choose to create a new
  EDE option that conveys the information in the "c", "s" and "j"
  fields encoded in the JSON object.



# Interoperation with RPZ Servers

This section discusses operation with an RPZ server {{RPZ}} that
indicates filtering with a NXDOMAIN response with the Recursion
Available bit cleared (RA=0).

When the DNS client supports this specification but the server does
not, the server will continue replying when a query is RPZ filtered
with NXDOMAIN and RA=0. An DNS client upgraded to support this
specification can continue to accept responses with NXDOMAIN and RA=0
from the RPZ server that does not support this specification.

When the DNS client supports this specification and the server
supports this specification, the client learns of the server's support
via {{?I-D.reddy-add-resolver-info}} and the client includes the EDE OPT
pseudo-RR in the query. This allows the server to differentiate
EDE-aware clients from EDE-unaware clients and respond appropriately.

# New Sub-Error Codes Definition

Subsequent documents defining new Sub-Error codes MUST be accepted by
IETF, MUST also provide a clear definition of the error code (in their
Reference text), and MUST also provide more characterization than the
{{RFC8914}} parent error.

This document defines new IANA-registered Sub-Error codes, below.

## Reserved {#policy-reserved}

  * Number: 0

  * Meaning: Reserved

  * Applicability: Blocked, Forged, Censored, Filtered

  * Reference: this sub-error code value MUST NOT be sent. If received, it has no meaning.

  * Change Controller: IETF


## Network Operator Policy {#policy-network}

  * Number: 5

  * Meaning: Network Operator Policy

  * Applicability: Blocked, Forged

  * Reference: Filtered according to policy determined by the operator of the local network 

  * Change Controller: IETF


## DNS Operator Policy {#policy-dns}

  * Number: 6

  * Meaning: DNS Operator Policy

  * Applicability: Blocked, Forged

  * Reference:  Filtered according to policy determined by the operator of the DNS server

  * Change Controller: IETF


# Examples

An example showing the nameserver at 'ns.example.net' that filtered a
DNS "A" record query for 'example.org' is shown in {{example-json}}.

~~~~~
{
  "c": [
    "tel:+358-555-1234567",
    "sips:bob@bobphone.example.com",
    "https://ticket.example.com?d=example.org&t=1650560748"
  ],
  "j": "malware present for 23 days",
  "s": 1,
  "o": "example.net Filtering Service"
}
~~~~~
{: #example-json title="JSON returned in EXTRA-TEXT field of Extended DNS Error response"}

In {{example-json-minified}} the same content is shown with minified JSON (no
whitespace, no blank lines) with '\\' line wrapping per {{?RFC8792}}.

~~~~~
 ============== NOTE: '\' line wrapping per RFC 8792 ===============

  {"c":["tel:+358-555-1234567","sips:bob@bobphone.example.com", \
  "https://ticket.example.com?d=example.org&t=1650560748"],"s":1, \
  "j":"malware present for 23 days","o":"example.net Filtering \
  Service"}
~~~~~
{: #example-json-minified title="Minified response"}

# Security Considerations

Security considerations in Section 6 of {{!RFC8914}} apply to this
document.

To minimize impact of active on-path attacks on the DNS channel, the
client validates the response as described in {{client-processing}}.

A client might choose to display the information in the "c", "j" and
"o" fields if and only if the encrypted resolver has sufficient
reputation, according to some local policy (e.g. user configuration,
administrative configuration, or a built-in list of respectable
resolvers). This limits the ability of a malicious encrypted resolver
to cause harm. If the client decides not to display the all of the
information in the EXTRA-TEXT field, it can be logged for diagnostics
purpose and the client can only display the resolver hostname that
blocked the domain, error description for the EDE code and the
suberror description for the "s'" field to the end-user.

When displaying the free-form text of "c" and "o", the browser SHOULD
NOT make any of those elements into actionable (clickable) links.

An attacker might inject (or modify) the EDE EXTRA-TEXT field with an
DNS proxy or DNS forwarder that is unaware of EDE. Such a DNS proxy or
DNS forwarder will forward that attacker-controlled EDE option. To
prevent such an attack, clients supporting this document MUST discard
the EDE option if their DNS server does not signal EDE support via
RESINFO {{?I-D.reddy-add-resolver-info}}. As recommended in
{{?I-D.reddy-add-resolver-info}}, RESINFO should be retrieved over an
encrypted DNS channel or integrity protected with DNSSEC.


# IANA Considerations {#IANA}

This document requests two IANA actions as described in the following subsections.

## New structured-dns-error Media Type

This document requests IANA to register the
"application/json+structured-dns-error" media type in the "Media
Types" registry {{IANA-MediaTypes}}. This registration follows the
procedures specified in {{!RFC6838}}:

~~~~~
   Type name: application

   Subtype name: json+structured-dns-error

   Required parameters: N/A

   Optional parameters: N/A

   Encoding considerations: as defined in Section NN of [RFCXXXX].

   Security considerations: See Section NNN of [RFCXXXX].

   Interoperability considerations: N/A

   Published specification: [RFCXXXX]

   Applications that use this media type: Section NNNN of [RFCXXXX].

   Fragment identifier considerations: N/A

   Additional information: N/A

   Person & email address to contact for further information: IETF,
      iesg@ietf.org

   Intended usage: COMMON

   Restrictions on usage: none

   Author: See Authors' Addresses section.

   Change controller: IESG

   Provisional registration?  No
~~~~~

## New Registry for SubError Codes

This document requests IANA to create a new registry, entitled "SubError Codes"
under "Domain Name System (DNS) Parameters, Extended DNS Error Codes"
registry {{IANA-DNS}}. The registration request for a new suberror codes MUST include the
following fields:

* Number: Wire format suberror code (range 0-255)

* Meaning: A short description of the error including reference to an IETF-approved document

* Applicability: Indicates which RFC8914 error codes apply to this sub-error code

* Reference: A pointer to the specification text

* Change Controller: Person or entity, with contact information if appropriate.

The SubError Code registry shall initially be populated with the
following suberror codes:

| Number | Meaning | RFC8914 error code applicability | Reference |  Change Controller |
|:------:|:--------|:---------------------------------|:----------|:------------------:|
| 0 | Reserved| Blocked, Forged, Censored, Filtered | {{policy-reserved}} | IETF |
| 1 | Malware | Blocked, Forged, Censored, Filtered | Section 5.5 of {{!RFC5901}} | IETF |
| 2 | Phishing | Blocked, Forged, Censored, Filtered | Section 5.5 of {{!RFC5901}} | IETF |
| 3 | Spam | Blocked, Forged, Censored, Filtered | Page 289 of {{?RFC4949}} | IETF |
| 4 | Spyware | Blocked, Forged, Censored, Filtered | Page 291 of {{!RFC4949}} | IETF |
| 5 | Network operator policy | Blocked, Forged | {{policy-network}} | IETF |
| 6 | DNS operator policy | Blocked, Forged | {{policy-dns}} | IETF |
{: #reg title='Initial SubError Code Rregistry'}

New entries in this registry are subject to an Expert Review
registration policy {{!RFC8126}}. The designated expert MUST ensure that
the Reference is stable and publicly available, and that it specifies
the suberror code and a short description. The reference may be
an individual Internet-Draft, or a document from any other source
with similar assurances of stability and availability.

Review requests are evaluated on the advice of one or more
designated experts. Criteria that should be applied by the designated
experts include determining whether the proposed registration
duplicates existing entries and whether the registration description
is sufficiently detailed and fits the purpose of this registry.
The designated experts will either approve or deny the registration
request, communicating this decision to IANA. Denials should
include an explanation and, if applicable, suggestions as to how
to make the request successful.

--- back

# Acknowledgements
{:numbered="false"}

Thanks to Vittorio Bertola, Wes Hardaker, Ben Schwartz, Erid Orth,
Viktor Dukhovni, Warren Kumari, Paul Wouters, John Levine, and Bob
Harold for the comments.
