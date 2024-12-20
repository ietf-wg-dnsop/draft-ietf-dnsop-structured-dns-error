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
workgroup: "DNS Operations Working Group"
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
    email: "danwing@gmail.com"
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

  Impl-1:
     title: "Use of DNS Errors To improve Browsing User Experience With network based malware protection"
     target: https://datatracker.ietf.org/meeting/116/materials/slides-116-dnsop-dns-errors-implementation-proposal-slides-116-dnsop-update-on-dns-errors-implementation-00
     date: 30-03-2023

--- abstract

DNS filtering is widely deployed for various reasons, including
network security. However, filtered DNS responses lack structured information
for end users to understand the reason for the filtering.
Existing mechanisms to provide explanatory details to end users cause harm
especially if the blocked DNS response is for HTTPS resources.

This document updates RFC 8914 by signaling client support for structuring the EXTRA-TEXT field of
the Extended DNS Error to provide details on the DNS filtering. Such
details can be parsed by the client and displayed, logged, or used for
other purposes.

--- middle

# Introduction

DNS filters are deployed for a variety of reasons, e.g., endpoint
security, parental filtering, and filtering required by law
enforcement. Network-based security solutions such as firewalls and
Intrusion Prevention Systems (IPS) rely upon network traffic
inspection to implement perimeter-based security policies and operate
by filtering DNS responses. In a home network, DNS filtering is used for the
same reasons as above and additionally for parental control. Internet
Service Providers (ISPs) typically block access to some DNS domains due to a
requirement imposed by an external entity (e.g., law enforcement
agency) also performed using DNS-based content filtering.

Users of DNS services that perform filtering may wish to receive more
explanatory information about such a filtering to resolve problems with the filter
-- for example to contact the administrator to allowlist a DNS domain that
was erroneously filtered or to understand the reason a particular
domain was filtered. With that information, a user can choose
to use another network, open a trouble ticket with the DNS administrator to
resolve erroneous filtering, log the information, etc.

For the DNS filtering mechanisms described in {{existing-techniques}}, the DNS
server can return extended error codes Blocked, Filtered, or
Forged Answer defined in {{Section 4 of !RFC8914}}. However, these codes
only explain that filtering occurred but lack detail for the user to
diagnose erroneous filterings.

No matter which type of response is generated (forged IP address(es),
NXDOMAIN or empty answer, even with an extended error code), the user
who triggered the DNS query has little chance to understand which
entity filtered the query, how to report a mistake in the filter, or
why the entity filtered it at all. This document describes a mechanism
to provide such detail.

One of the other benefits of the approach described in this document is to eliminate the need to
"spoof" block pages for HTTPS resources. This is achieved since
clients implementing this approach would be able to display a
meaningful error message, and would not need to connect to such a
block page. This approach thus avoids the need to install a local root
certificate authority on those IT-managed devices.

This document describes a format for computer-parsable data in the
EXTRA-TEXT field of {{!RFC8914}}. It updates {{Section 2 of !RFC8914}} which
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
that responds to questions.

"Encrypted DNS" refers to any encrypted scheme to convey DNS messages,
for example, DNS-over-HTTPS {{?RFC8484}}, DNS-over-TLS {{?RFC7858}}, or
DNS-over-QUIC {{?RFC9250}}.

The document refers to an Extended DNS Error (EDE) using its purpose, not its
INFO-CODE as per Table 3 of {{!RFC8914}}. "Forged Answer",
"Blocked", and "Filtered" are thus used to refer to "Forged Answer (4)",
"Blocked (15)", and "Filtered (17)".

# DNS Filtering Techniques and Their Limitations {#existing-techniques}

DNS responses can be filtered by sending, e.g., a bogus (also called
"forged") response, NXDOMAIN error, or empty answer. Also, clients can be informed that filtering has occured by sending an
Extended DNS Error code defined in {{!RFC8914}}. Each of these
methods have advantages and disadvantages that are discussed below:

1. The DNS response is forged to provide a list of IP addresses that
points to an HTTP(S) server alerting the end user about the reason for
blocking access to the requested domain (e.g., malware). When an
HTTP(S) enabled domain name is blocked, the network security device
(e.g., Customer Premises Equipment (CPE) or firewall) presents a block page instead of the HTTP
response from the content provider hosting that domain. If an HTTP
enabled domain name is blocked, the network security device intercepts
the HTTP request and returns a block page over HTTP. If an HTTPS
enabled domain is blocked, the block page is also served over
HTTPS. In order to return a block page over HTTPS, man in the middle
(MITM) is enabled on endpoints by generating a local root certificate
and an accompanying (local) public/private key pair. The local root
certificate is installed on the endpoint while the network security
device stores a copy of the private key. During the TLS handshake,
the on-path network security device modifies the certificate provided by the
server and (re)signs it using the private key from the local root
certificate.

   * However, this approach is ineffective when DNSSEC is deployed given that DNSSEC
     ensures the integrity and authenticity of DNS responses, preventing forged DNS
     responses from being accepted.

   * The HTTPS server will have access to the client's IP address and the hostname being requested.
     This information will be sensitive, as it will expose the user's identity and the
     domain name that a user attempted to access.

   * Configuring a local root certificate on endpoints is
     not a viable option in several deployments like home networks,
     schools, Small Office/Home Office (SOHO), or Small/Medium
     Enterprise (SME). In these cases, the typical behavior is that
     the filtered DNS response points to a server that will display
     the block page. If the client is using HTTPS (via a web browser or
     another application) this results in a certificate validation
     error which gives no information to the end-user about the reason
     for the DNS filtering.

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
the DNS lookup to terminate in failure. However, this approach does not
work with DNSSEC. In scenarios without DNSSEC, an end user does not know why
a domain name cannot be reached and may repeatedly try to reach that domain name
but without success. Frustrated, an end user may use insecure connections
to reach the domain name, potentially compromising both security and privacy.

3. The extended error codes Blocked and Filtered defined in
{{Section 4 of !RFC8914}} can be returned by a DNS server to provide
additional information about the cause of a DNS error.

4. These extended error codes do not suffer from the limitations
discussed in bullets (1) and (2), but the user still does not know the
exact reason nor is aware of the exact entity blocking the
access to the domain. For example, a DNS server may block access to a
domain based on the content category such as "Malware" to protect the
endpoint from malicious software, "Phishing" to prevent the user from
revealing sensitive information to the attacker, etc. A user may need to
know the contact details of the IT/InfoSec team to raise a complaint.

# I-JSON in EXTRA-TEXT Field {#name-spec}

DNS servers that are compliant with this specification and have received an indication that the client also supports this specification as per {{client-request}} send data in the EXTRA-TEXT field {{!RFC8914}} encoded using the Internet JSON (I-JSON)
message format {{!RFC7493}}.

> Note that {{!RFC7493}} was based on {{!RFC7159}}, but {{!RFC7159}} was replaced by {{?RFC8259}}.

This document defines the following JSON names:

c: (contact)
: The contact details of the IT/InfoSec team to report mis-classified
DNS filtering. This information is important for transparency and also to ease unblocking a legitimate domain name that got blocked due to wrong classification.
: This field is structured as an array of contact URIs, using 'tel' {{!RFC3966}} or 'sips' {{!RFC5630}} or  'mailto' {{!RFC3966}} schemes. At least one contact URI MUST be included.

: New contact URI schemes may be added to the IANA registry following the instructions in {{IANA-Contact}}.
: This field is mandatory.

j: (justification)
: 'UTF-8'-encoded {{!RFC5198}} textual justification for this particular
DNS filtering. The field should be treated only as diagnostic
information for IT staff.
: Whether the information provided in the "j" name is meaningful or considered as garbage data
(including empty values) is local to each IT teams. Returning garbage data
would indicate that a DNS server is misbehaving. Note also that the provided
justification is useful for cross-validation with another DNS server.
: This field is mandatory.

s: (suberror)
: The suberror code for this particular DNS filtering.
: This field is optional.

o: (organization)
: 'UTF-8'-encoded human-friendly name of the organization that filtered this particular DNS query.
: This field is optional.

l: (language)
: The "l" field indicates the language used for the JSON-encoded "j" and "o" fields.  The value of this field MUST conform to the
  language tag syntax specified in {{Section 2.1 of !RFC5646}}.
: This field is optional but RECOMMENDED to aid in localization.

New JSON names can be defined in the IANA registry introduced in {{IANA-Names}}. Such names MUST
consist only of lower-case ASCII characters, digits, and hyphen-minus (that
is, Unicode characters U+0061 through 007A, U+0030 through U+0039, and
U+002D). Also, these names MUST be 63 characters or shorter and it is
RECOMMENDED they be as short as possible.

The text in the "j" and "o" names can include international
characters. The text will be in natural language, chosen by the DNS administrator
to match its expected audience.

To reduce DNS message size the generated JSON SHOULD be as short as
possible: short domain names, concise text in the values for the "j"
and "o" names, and minified JSON (that is, without spaces or line
breaks between JSON elements).

The JSON data can be parsed to display to the user, logged, or
otherwise used to assist the end-user or IT staff with troubleshooting
and diagnosing the cause of the DNS filtering.

> An alternate design for conveying the suberror would be to define new EDE codes for these errors. However, such design is suboptimal because it requires replicating an error code for each EDE code to which the suberror applies (e.g., "Malware" suberror in {{reg}} would consume three EDE codes).

# Protocol Operation

## Client Generating Request {#client-request}

When generating a DNS query the client includes the EDE option ({{Section 2 of !RFC8914}}) in the OPT pseudo-RR {{!RFC6891}} to
elicit the EDE option in the DNS response. It MUST use an
OPTION-LENGTH of 2, the INFO-CODE field set to "0"
(Other Error), and an empty EXTRA-TEXT field.  This signal indicates
that the client desires that the server responds in accordance with
the present specification.

## Server Generating Response {#server-response}

When the DNS server filters its DNS response to a
query (e.g., A or AAAA record query), the DNS response MAY contain an empty answer, NXDOMAIN, or (less
ideally) forged response, as desired by the DNS
server. In addition, if the query contained the OPT pseudo-RR the DNS
server MAY return more detail in the EXTRA-TEXT field as described in
{{client-processing}}.

Servers MAY decide to return small TTL values in filtered DNS
responses (e.g., 2 seconds) to handle domain category and reputation
updates. Short TTLs allow for quick adaptation to dynamic changes in domain filtering decisions,
but can result in increased query traffic. In cases where updates are less frequent,
TTL values of 30 to 60 seconds MAY provide a better balance, reducing server load while
still ensuring reasonable flexibility for updates.

Because the DNS client signals its EDE support ({{client-request}})
and because EDE support is signaled via a non-cached OPT resource
record ({{Section 6.2.1 of ?RFC6891}}) the EDE-aware DNS server can
tailor its filtered response to be most appropriate to that client's
EDE support.  If EDE support is signaled in the query as per {{client-request}}, the server MUST
NOT return the "Forged Answer" extended error code because the client
can take advantage of EDE's more sophisticated error reporting (e.g.,
"Filtered", "Blocked").  Continuing to send "Forged
Answer" even to an EDE-supporting client will cause the persistence of
the drawbacks described in {{existing-techniques}}.

## Client Processing Response {#client-processing}

On receipt of a DNS response with an EDE option from a
DNS responder, the following ordered actions are performed on the EXTRA-TEXT
field:

* Servers which don't support this specification might use plain text
  in the EXTRA-TEXT field. Requestors SHOULD properly handle
  both plaintext and JSON text in the EXTRA-TEXT field. The requestor verifies that
  the field contains valid JSON. If not, the requestor MUST consider
  the server does not support this specification and stop processing
  rest of the actions defined in this section, but may instead choose
  to treat EXTRA-TEXT as per {{!RFC8914}}.

* The response MUST be received over an encrypted DNS channel. If not,
  the requestor MUST discard data in the EXTRA-TEXT field.

* The DNS response MUST also contain an extended error code of
  "Blocked by Upstream Server", "Blocked" or "Filtered" {{!RFC8914}}, otherwise
  the EXTRA-TEXT field is discarded.

* If either of the mandatory JSON names "c" and "j" are missing or
  have empty values in the EXTRA-TEXT field, the entire JSON is
  discarded.

* If the "c" field contains any URI scheme not registered in the
  {{IANA-Contact}} registry, it MUST be discarded.

* If a DNS client has enabled opportunistic privacy profile ({{Section 5
  of !RFC8310}}) for DoT, the DNS client will either fall back to an
  encrypted connection without authenticating the DNS server provided
  by the local network or fall back to clear text DNS, and cannot
  exchange encrypted DNS messages. Both of these fallback mechanisms
  adversely impact security and privacy. If the DNS client has enabled
  opportunistic privacy profile for DoT and the identity of the DNS server
  cannot be verified but the connection is encrypted, the DNS client MUST
  ignore the "c", "j", and "o" fields but MAY process the "s" field
  and other parts of the response.

* Opportunistic discovery {{?I-D.ietf-add-ddr}}, where only the IP address is
  validated, the DNS client MUST ignore the "c", "j", and "o" fields
  but MAY process the "s" field and other parts of the response.

* If a DNS client has enabled strict privacy profile ({{Section 5 of !RFC8310}}) for DoT, the DNS client requires an encrypted connection
  and successful authentication of the DNS server. In doing so, this mitigates both
  passive eavesdropping and client redirection (at the expense of
  providing no DNS service if an encrypted, authenticated connection
  is not available). If the DNS client has enabled strict privacy
  profile for DoT, the DNS client MAY process the EXTRA-TEXT field of the
  DNS response.

* The DNS client MUST ignore any other JSON names that it does not support.

* When a forwarder receives an EDE option, whether or not (and how) to
  pass along JSON information in the EXTRA-TEXT on to their client is
  implementation dependent {{?RFC5625}}. Implementations MAY choose to
  not forward the JSON information, or they MAY choose to create a new
  EDE option that conveys the information in the "c", "s", and "j"
  fields encoded in the JSON object.

* The application that triggered the DNS request may have a local policy to override the contact information
 (e.g., redirect all complaint calls to a single contact point). In such a case, the content of the
 "c" attribute can be ignored.


> Note that the strict and opportunistic privacy profiles as defined in {{!RFC8310}} only apply to DoT; there has been
no such distinction made for DoH.


# New Sub-Error Codes Definition

The document defines the following new IANA-registered Sub-Error codes.

## Reserved {#policy-reserved}

  * Number: 0

  * Meaning: Reserved. This sub-error code value MUST NOT be sent. If received, it has no meaning.

  * Applicability: This code should never be used.

  * Reference: This-Document

  * Change Controller: IETF


## Network Operator Policy {#policy-network}

  * Number: 5

  * Meaning: Network Operator Policy. The code indicates that the request was filtered according to a policy imposed by the operator of the local network (where local network is a relative term, e.g., it may refer to a Local Area Network or to the network of the ISP selected by the user).

  * Applicability: Blocked

  * Reference: This-Document

  * Change Controller: IETF


## DNS Operator Policy {#policy-dns}

  * Number: 6

  * Meaning: DNS Operator Policy. The code indicates that the request was filtered according to policy determined by the operator of the DNS server. This is different from the "Network Operator Policy" code when a third-party DNS resolver is used.

  * Applicability: Blocked

  * Reference:  This-Document

  * Change Controller: IETF

# Extended DNS Error Code TBA1 - Blocked by Upstream DNS Server

The DNS server (e.g., a DNS forwarder) is unable to respond to the request
because the domain is on a blocklist due to an internal security policy
imposed by an upstream DNS server. This error code
is useful in deployments where a network-provided DNS forwarder
is configured to use an external resolver that filters malicious
domains. Typically, when the DNS forwarder receives a Blocked (15) error code from the upstream DNS server, it will replace it with "Blocked by Upstream DNS Server" (TBA1) before forwarding the reply to the DNS client.

# Examples

An example showing the nameserver at 'ns.example.net' that filtered a
DNS "A" record query for 'example.org' is provided in {{example-json}}.

~~~~~
{
  "c": [
    "tel:+358-555-1234567",
    "sips:bob@bobphone.example.com"
  ],
  "j": "malware present for 23 days",
  "s": 1,
  "o": "example.net Filtering Service"
  "l": "tzm",
}
~~~~~
{: #example-json title="JSON Returned in EXTRA-TEXT Field of Extended DNS Error Response"}

In {{example-json-minified}} the same content is shown with minified JSON (no
whitespace, no blank lines) with ```'\'``` line wrapping per {{?RFC8792}}.

~~~~~
{::include-fold68left0 ./examples/minified.json}
~~~~~
{: #example-json-minified title="Minified Response"}

# Security Considerations {#security}

Security considerations in {{Section 6 of !RFC8914}} apply to this
document, except the guard against using EDE content to alter DNS protocol
processing. The guard is relaxed in the current specification as it mandates
encryption and recommends the use of an authenticated connection to the DNS
server, while {{!RFC8914}} assumes that EDE information is unauthenticated
and sent over clear text.

To minimize impact of active on-path attacks on the DNS channel, the
client validates the response as described in {{client-processing}}.

A client might choose to display the information in the "c", "j", and
"o" fields if and only if the encrypted resolver has sufficient
reputation, according to some local policy (e.g., user configuration,
administrative configuration, or a built-in list of respectable
resolvers). This limits the ability of a malicious encrypted resolver
to cause harm. For example, an end user can use the details in the "c" field to contact an attacker
to solve the problem of being unable to reach a domain. The attacker can mislead the end user to
install malware or spyware to compromise the device security posture or mislead the end user to reveal
personal data.
If the client decides not to display all of the
information in the EXTRA-TEXT field, it can be logged for diagnostics
purpose and the client can only display the resolver hostname that
blocked the domain, error description for the EDE code and the
suberror description for the "s" field to the end-user.

When displaying the free-form text of "j" and "o", the browser MUST
NOT make any of those elements into actionable (clickable) links and these
fields need to be rendered as text, not as HTML. The contact details of "c" can be made
into clickable links to provide a convenient way for users to initiate, e.g., voice calls. The client might
choose to display the contact details only when the identity of the DNS server is verified.

An attacker might inject (or modify) the EDE EXTRA-TEXT field with a
DNS proxy or DNS forwarder that is unaware of EDE. Such a DNS proxy or
DNS forwarder will forward that attacker-controlled EDE option.  To
prevent such an attack, clients can be configured to process EDE from
explicitly configured DNS servers or utilize RESINFO
{{?I-D.ietf-add-resolver-info}}.

# IANA Considerations {#IANA}

This document requests four IANA actions as described in the following subsections.

> Note to the RFC Editor: Please replace RFCXXXX with the RFC number assigned to this document and "TBA1" with the value assigned by IANA.

## Media Type Registration

This document requests IANA to register the
"application/json+structured-dns-error" media type in the "Media
Types" registry {{IANA-MediaTypes}}. This registration follows the
procedures specified in {{!RFC6838}}:

~~~~~
   Type name: application

   Subtype name: json+structured-dns-error

   Required parameters: N/A

   Optional parameters: N/A

   Encoding considerations: as defined in Section 4 of RFCXXXX.

   Security considerations: See Section 10 of RFCXXXX.

   Interoperability considerations: N/A

   Published specification: RFCXXXX

   Applications that use this media type: Section 4 of RFCXXXX.

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

##  New Registry for JSON Names {#IANA-Names}

This document requests IANA to create a new registry, entitled "EXTRA-TEXT JSON Names"
under "Domain Name System (DNS) Parameters, Extended DNS Error Codes"
registry {{IANA-DNS}}. The registration request for a new JSON name must include the
following fields:

JSON Name:
: Specifies the name of an attribute that is present in the JSON data enclosed in EXTRA-TEXT field. The name must follow the guidelines in {{name-spec}}.

Short description:
: Includes a short description of the requested JSON name.

Mandatory (Y/N?):
: Indicates whether this attribute is mandatory or optional.

Specification:
: Provides a pointer to the reference document that specifies the attribute.

The registry is initially populated with the following values:

| JSON Name | Full JSON Name | Description                      | Mandatory |  Specification |
|:---------:|:---------------|:---------------------------------|:----------|:------------------:|
| c | contact| The contact details of the IT/InfoSec team to report mis-classified DNS filtering | Y | {{name-spec}} of RFCXXXX |
| j | justification | UTF-8-encoded {{!RFC5198}} textual justification for a particular DNS filtering | Y | {{name-spec}} of RFCXXXX |
| s | suberror | the suberror code for this particular DNS filtering | N | {{name-spec}} of RFCXXXX |
| o | organization | UTF-8-encoded human-friendly name of the organization that filtered this particular DNS query | N | {{name-spec}} of RFCXXXX |
| l | language     | Indicates the language of the "j" and "o" fields as defined in {{!RFC5646}} | No | {{name-spec}} of RFCXXXX |
{: #reg-names title='Initial JSON Names Registry'}

New JSON names are registered via IETF Review ({{Section 4.8 of !RFC8126}}).


## New Registry for Contact URI Scheme {#IANA-Contact}

This document requests IANA to create a new registry, entitled "Contact URI Schemes"
under "Domain Name System (DNS) Parameters, Extended DNS Error Codes"
registry {{IANA-DNS}}. The registration request for a new Contact URI scheme has to include the
following fields:

* Name: URI scheme name.

* Meaning: Provides a short description of the scheme.

* Reference: Provides a pointer to an IETF-approved specification that defines
  the URI scheme.

* Change Controller: Indicates the person or entity, with contact information if appropriate.

The Contact URI scheme registry is initially be populated with the
following schemes:

| Name      | Meaning           | Reference     | Change Controller |
|:---------:|:------------------|:-------------:|:-----------------:|
| sips      | SIP Call           | {{!RFC5630}} |     IETF          |
| tel       | Telephone Number   | {{!RFC3966}} |     IETF          |
| mailto    | Internet mail      | {{!RFC6068}} |     IETF          |

{: #reg-contact='Initial Contact URI Schemes Registry'}


New Contact URI schemes are registered via IETF Review ({{Section 4.8 of !RFC8126}}).

## New Registry for DNS SubError Codes

This document requests IANA to create a new registry, entitled "SubError Codes"
under "Domain Name System (DNS) Parameters, Extended DNS Error Codes"
registry {{IANA-DNS}}. The registration request for a new suberror codes MUST include the
following fields:

* Number: Is the wire format suberror code (range 0-255).

* Meaning: Provides a short description of the sub-error.

* Applicability: Indicates which RFC8914 error codes apply to this sub-error code.

* Reference: Provides a pointer to an IETF-approved specification that registered
  the code and/or an authoritative specification that describes the
  meaning of this code.

* Change Controller: Indicates the person or entity, with contact information if appropriate.

The SubError Code registry is initially be populated with the
following suberror codes:

| Number | Meaning | RFC8914 error code applicability | Reference |  Change Controller |
|:------:|:--------|:---------------------------------|:----------|:------------------:|
| 0 | Reserved| Not used | {{policy-reserved}} of this document | IETF |
| 1 | Malware | "Blocked", "Blocked by Upstream Server", "Filtered" | Section 5.5 of {{!RFC5901}} | IETF |
| 2 | Phishing | "Blocked", "Blocked by Upstream Server", "Filtered" | Section 5.5 of {{!RFC5901}} | IETF |
| 3 | Spam | "Blocked", "Blocked by Upstream Server", "Filtered" | Page 289 of {{?RFC4949}} | IETF |
| 4 | Spyware | "Blocked", "Blocked by Upstream Server", "Filtered" | Page 291 of {{!RFC4949}} | IETF |
| 5 | Network operator policy | "Blocked" | {{policy-network}} of this document | IETF |
| 6 | DNS operator policy | "Blocked" | {{policy-dns}} of this document | IETF |
{: #reg title='Initial SubError Code Registry'}

New SubError Codes are registered via IETF Review ({{Section 4.8 of !RFC8126}}).

## New Extended DNS Error Code

IANA is requested to assign the following Extended DNS Error code from the "Domain Name System (DNS) Parameters, Extended DNS Error Codes"
registry {{IANA-DNS}}:

| INFO-CODE | Purose                           | Reference |
|:---------:|:---------------------------------|:---------:|
| TBA1      | Blocked by Upstream Server       | RFCXXXX   |
{: #reg-ede title='New DNS Error Code'}

--- back

# Interoperation with RPZ Servers

This appendix provides a non-normative guidance for operation with an Response Policy Zones (RPZ) server {{RPZ}} that
indicates filtering with a NXDOMAIN response with the Recursion
Available bit cleared (RA=0). This guidance is provided to ease interoperation with RPZ.

When a DNS client supports this specification, it includes the
EDE option in its DNS query.

If the server does not support this specification and is performing
RPZ filtering, the server ignores the EDE option in the DNS query and
replies with NXDOMAIN and RA=0. The DNS client can continue to accept
such responses.

If the server does support this specification and is performing RPZ
filtering, the server can use the EDE option in the query to identify
an EDE-aware client and respond appropriately (that is, by generating
a response described in {{server-response}}) as NXDOMAIN and RA=0
are not necessary when generating a response to such a client.

# Implementation Status

> Note to the RFC Editor: please remove this appendix prior publication.

At IETF#116, Gianpaolo Scalone (Vodafone) and Ralf Weber (Akamai) presented an implementation of this specification. More details can be found at {{Impl-1}}.



# Acknowledgements
{:numbered="false"}

Thanks to Vittorio Bertola, Wes Hardaker, Ben Schwartz, Erid Orth,
Viktor Dukhovni, Warren Kumari, Paul Wouters, John Levine, Bob
Harold, Mukund Sivaraman, Stephane Bortzmeyer, Gianpaolo Angelo Scalone, and Daniel Migault for the comments.

Thanks to Ralf Weber and Gianpaolo Scalone for sharing details about their implementation.

Thanks Di Ma and Matt Brown for the DNS directorate reviews, and Joseph Salowey for the Security directorate review.



