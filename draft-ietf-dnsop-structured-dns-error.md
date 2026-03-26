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
  IANA-Enterprise:
     title: "Private Enterprise Numbers (PENs)"
     target: https://www.iana.org/assignments/enterprise-numbers/
     date: false

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

End users or network administrators leveraging DNS services that perform filtering may wish to receive more
explanatory information about such a filtering to resolve problems with the filter
-- for example to contact the DNS service administrator to allowlist a DNS domain that
was erroneously filtered or to understand the reason a particular
domain was filtered. With that information, they can choose
to use another network, open a trouble ticket with the DNS service administrator to
resolve erroneous filtering, log the information, etc.

For the DNS filtering mechanisms described in {{existing-techniques}}, the DNS
server can return extended error codes Blocked, Filtered, Censored, or
Forged Answer defined in {{Section 4 of !RFC8914}}. However, these codes
only explain that filtering occurred but lack detail for the user to
diagnose erroneous filtering.

No matter which type of response is generated (forged IP address(es),
NXDOMAIN or empty answer, even with an extended error code), the end user
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

This document describes a format for machine-readable data in the
EXTRA-TEXT field of {{!RFC8914}}. It updates {{Section 2 of !RFC8914}} which
says the information in EXTRA-TEXT field is intended for human
consumption (not automated parsing).

This document does not recommend DNS filtering but provides a
mechanism for better transparency to explain to the end users why some DNS
queries are filtered.



# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document uses terms defined in DNS Terminology {{!RFC9499}}.

"Encrypted DNS" refers to any encrypted scheme to convey DNS messages,
for example, DNS-over-HTTPS {{?RFC8484}}, DNS-over-TLS {{?RFC7858}}, or
DNS-over-QUIC {{?RFC9250}}.

The document refers to an Extended DNS Error (EDE) using its purpose, not its
INFO-CODE as per Table 3 of {{!RFC8914}}. "Forged Answer",
"Blocked", "Censored", and "Filtered" are thus used to refer to "Forged Answer (4)",
"Blocked (15)", "Censored (16)",and "Filtered (17)".

In this document, "client security policy evaluation" refers to implementation-defined
decision-making performed by the DNS client or consuming application (e.g., web
browser) to
determine how, or whether, structured error information is used, displayed,
or acted upon.

Structured DNS Error (SDE) is an EDNS(0) option indicating support for structured encoding of the EXTRA-TEXT field.

# DNS Filtering Techniques and Their Limitations {#existing-techniques}

DNS responses can be filtered by sending, e.g., a bogus (also called
"forged") response, NXDOMAIN error, or empty answer. Also, clients can be informed that filtering occurred by sending an
Extended DNS Error code defined in {{!RFC8914}}. Each of these
methods have advantages and disadvantages that are discussed below:

1. The DNS response is forged to provide a list of IP addresses that
points to an HTTP(S) server alerting the end user about the reason for
blocking access to the requested domain (e.g., malware). If the host component {{?RFC3986}}
of an HTTP URL is blocked, the network security device
(e.g., Customer Premises Equipment (CPE) or firewall) presents a block page instead of the HTTP
response from the content provider hosting that domain. This works succesfully with HTTP.
<br/><br/>
  If this is an HTTPS URL, the network security device attempts to serve the block page over HTTPS.  In order to return a block page over HTTPS, the network security device uses a locally
generated root certificate and corresponding key pair. The local root certificate is
installed on the endpoint while the network security device stores a copy of the private key.
During the TLS handshake, the on-path network security device modifies the certificate
provided by the server and (re)signs it using the private key from the local root
certificate.

   * In deployments where DNSSEC is used, this approach becomes ineffective because DNSSEC
     ensures the integrity and authenticity of DNS responses, preventing forged DNS
     responses from being accepted.

   * The HTTPS server hosted on the network security device will have access to the client's IP address and the
     hostname being requested. This information will be sensitive, as it will expose the end user's identity and the
     domain name that a end user attempted to access.

   * Configuring a local root certificate on endpoints is
     not a viable option in several deployments like home networks,
     schools, Small Office/Home Office (SOHO), or Small/Medium
     Enterprise (SME). In these cases, the typical behavior is that
     the filtered DNS response points to a server that will display
     the block page. If the client is using HTTPS (via a web browser or
     another application) this results in a certificate validation
     error which gives no information to the end user about the reason
     for the DNS filtering.

   * Enterprise networks do not always assume that all the connected devices
     are managed by the IT team or Mobile Device Management (MDM)
     devices, especially in the quite common Bring Your Own Device
     (BYOD) scenario. In addition, the local root certificate cannot
     be installed on IoT devices without a device management tool.

   * An end user does not know why the connection was prevented and,
     consequently, may repeatedly try to reach the domain but with no
     success. Frustrated, the end user may switch to an alternate
     network that offers no DNS filtering against malware and
     phishing, potentially compromising both security and
     privacy. Furthermore, certificate errors train end users to click
     through certificate errors, which is a bad security practice. To
     eliminate the need for an end user to click through certificate
     errors, an end user may manually install a local root certificate
     on a host device. Doing so, however, is also a bad security
     practice as it creates a security vulnerability that may be
     exploited by a MITM attack. When a manually installed local root
     certificate expires, the end user has to (again) manually install the
     new local root certificate.

2. The DNS response is forged to provide an NXDOMAIN answer, causing the DNS lookup to fail. This approach is incompatible with DNSSEC when the client performs validation, as the forged response will fail DNSSEC checks. However, in deployments where the client relies on the DNS server to perform DNSSEC validation, a filtering DNS server can forge an NXDOMAIN response for a valid domain, and the client will trust it. This undermines the integrity guarantees of DNSSEC, as the client has no way to distinguish between a genuine and a forged response. Further, the end user may not understand why a domain cannot be reached and may repeatedly attempt access without success. Frustrated, the end user may resort to using insecure methods to reach the domain, potentially compromising both security and privacy.

3. The extended error codes Blocked and Filtered defined in
{{Section 4 of !RFC8914}} can be returned by a DNS server to provide
additional information about the cause of a DNS error.
These extended error codes do not suffer from the limitations
discussed in bullets (1) and (2), but the user still does not know the
exact reason nor is aware of the exact entity blocking the
access to the domain. For example, a DNS server may block access to a
domain based on the content category such as "Malware" to protect the
endpoint from malicious software, "Phishing" to prevent the end user from
revealing sensitive information to the attacker, etc. A end user may need to
know the contact details of the IT/InfoSec team to raise a complaint.
Further, the information conveyed by {{!RFC8914}} is intended for
diagnostic purposes and is not structured for automated processing,
localization, or extensibility. This document defines a structured,
machine-readable format for conveying such details in the EXTRA-TEXT
field, enabling clients to process the information programmatically
and present it to end users (e.g., with localization support), while
allowing for extensibility and more granular, client security
policy-driven handling of the information. This specification requires
that clients only act upon such information when it is received
over an integrity-protected DNS response.

# I-JSON in EXTRA-TEXT Field {#name-spec}

DNS servers that are compliant with this specification and have received an indication that the client also supports this specification as per {{client-request}} send data in the EXTRA-TEXT field {{!RFC8914}} as a JSON object encoded using the Internet JSON (I-JSON) message format {{!RFC7493}}.

This document defines the following JSON names:

c: (contact)
: The contact details of the IT/InfoSec team to report misclassified
DNS filtering. This information is important for transparency and also to ease unblocking a legitimate domain name that got blocked due to wrong classification.
: The field is a JSON array of contact URIs. When multiple contact details are provided, each contact URI is
represented as a separate array element in the JSON array.
: Contact URIs conveyed in the "c" field MUST use URI schemes registered in {{IANA-Contact}}.
: This field is optional.

j: (justification)
: 'UTF-8'-encoded {{!RFC5198}} human-readable explanation for the DNS
filtering decision.
: This field is particularly useful when no applicable sub-error code
is defined or provided for the returned Extended DNS Error.
: The information conveyed in this field MUST NOT be used as input to
automated processing that affects security policy enforcement or DNS
protocol behavior.
: The DNS client determines, according to its client security policy,
whether the contents of this field are displayed to the end user,
logged, or ignored.
: Returning non-UTF-8 data, syntactically invalid content, or
deliberately meaningless values (including empty strings) indicates that
a DNS server is misbehaving.
: This field is optional.

s: (sub-error)
: An integer representing the sub-error code for this particular DNS filtering case.
: The integer values are defined in the IANA-managed registry for DNS Sub-Error Codes in {{IANA-SubError}}.
: This field is optional.

o: (organization)
: 'UTF-8'-encoded human-friendly name of the organization that filtered this particular DNS query.
: This field is optional.

l: (language)
: The "l" field indicates the language used for the JSON-encoded "j" and "o" fields.  The value of this field MUST conform to the
  language tag syntax specified in {{Section 2.1 of !RFC5646}}.
: This field is optional but RECOMMENDED to aid in localization.

The text in the "j" and "o" names can include international
characters. The text will be in natural language, chosen by the DNS administrator
to match its expected audience.

If the client supports diagnostic interfaces, it MAY use the "l" field to identify
the language of the "j" text and optionally translate it.

The "o" field MAY be displayed to end users, subject to the conditions described in {{security}}.
If the text is in a language not understood by the end user, the "l" field can be used
to identify the language and support translation into the end user's preferred language.

To avoid exceeding the maximum EDNS0 size {{?RFC9715}} the generated JSON values SHOULD be as short as
possible: short domain names, concise text in the values for the "j"
and "o" names, and minified JSON (that is, without spaces or line
breaks between JSON elements).

The JSON data can be parsed to display to the user, logged, or
otherwise used to assist troubleshooting and diagnosis of DNS filtering.

The sub-error codes provide a structured way to communicate more detailed and precise description of the cause of an error (e.g., distinguishing between malware-related blocking and phishing-related blocking under the general blocked error).

> An alternate design for conveying the sub-error would be to define new EDE codes for these errors. However, such design is suboptimal because it requires replicating an error code for each EDE code to which the sub-error applies (e.g., "Malware" sub-error in {{reg}} would consume three EDE codes).

New JSON names MUST consist only of lower-case ASCII characters, digits,
and hyphen-minus (that is, Unicode characters U+0061 through 007A,
U+0030 through U+0039, and U+002D). Also, these names MUST be 63
characters or shorter and it is RECOMMENDED they be as short as
possible to reduce contribution to exceeding maximum EDNS0 response
size {{?RFC9715}}.




# Protocol Operation

## Client Generating Request {#client-request}

When generating a DNS query, a client that supports this specification
SHOULD include the Structured DNS Error (SDE) option defined in {{SDE}}, unless instructed by local policy otherwise.

The presence of the SDE option indicates that the client desires the
DNS server to include an EDE option in the DNS response when DNS
filtering is performed, and that any data conveyed in the EXTRA-TEXT
field of the EDE option is encoded and processed in accordance with
this specification.

## Server Generating Response {#server-response}

When the DNS server filters its DNS response to a
query (e.g., A or AAAA resource record query), the DNS response MAY contain an empty answer, NXDOMAIN, or (less
ideally) forged response, as desired by the DNS
server.

If the query contained the SDE EDNS option ({{client-request}}), and the DNS server returns an Extended DNS Error code of "Blocked", "Filtered", "Censored", or "Blocked by Upstream DNS Server", the DNS server SHOULD include additional detail in the EXTRA-TEXT field encoded as structured and machine-readable data in accordance with this specification, unless configured otherwise. If including the additional detail would cause the response to exceed the EDNS0 size {{?RFC9715}} (and thus setting TC=1), it SHOULD be omitted. In deployments using DNS-over-TLS, DNS-over-HTTPS, or DNS-over-QUIC, transport size limitations are unlikely to necessitate omission of structured data in the EXTRA-TEXT field.

If the SDE option was not present in the DNS request, the DNS server MUST process the request in accordance with {{!RFC8914}} and MUST NOT assume that the client supports this specification. This preserves compatibility with clients and servers that implement {{!RFC8914}} but do not support this specification.

Servers MAY decide to return small TTL values in filtered DNS
responses (e.g., 10 seconds) to handle domain category and reputation
updates. Short TTLs allow for quick adaptation to dynamic changes in domain filtering decisions,
but can result in increased query traffic. In cases where updates are less frequent,
TTL values of 30 to 60 seconds MAY provide a better balance, reducing server load while
still ensuring reasonable flexibility for updates.

If the query includes the SDE option as per {{client-request}}, the server MUST
NOT return the "Forged Answer" extended error code because the client
can take advantage of EDE's more sophisticated error reporting (e.g.,
"Filtered", "Blocked").  Continuing to send "Forged
Answer" even to an EDE-supporting client will cause the persistence of
the drawbacks described in {{existing-techniques}}.

When the "Censored" extended error code is included in the DNS response,
the "c", "j", "o", and "l" fields may be conveyed in the EXTRA-TEXT field.
The sub-error codes defined in this specification are not applicable to
the "Censored" extended error code and MUST NOT be used in conjunction with it.
Future specifications may update this behavior by defining sub-error codes
applicable to "Censored".

## Client Processing Response {#client-processing}

On receipt of a DNS response with an EDE option from a
DNS server, the following ordered actions are performed on the EXTRA-TEXT
field:

1. If the integrity of the DNS response is not guaranteed, the
   DNS client MUST NOT act upon data in the EXTRA-TEXT field, as the data
   is vulnerable to
   modification by an on-path attacker. An attacker can inject or
   modify a structured DNS error response in transit without detection,
   enabling fabrication of filtering information (e.g., misleading contact
   information or false resolver identity information) that appears to
   originate from the resolver. The data MAY be retained for diagnostic or
   client security policy evaluation purposes.

2. Servers that do not support this specification might use plain text in the
   EXTRA-TEXT field. DNS clients SHOULD handle both plaintext and structured content. The client attempts to parse the EXTRA-TEXT field as I-JSON. If parsing fails or the content is not valid I-JSON, the client MUST treat the data as invalid, MUST NOT process it according to this specification. The client MAY instead process the EXTRA-TEXT field as unstructured text as specified in {{!RFC8914}}.

3. The DNS response MUST also contain an extended error code of
   "Blocked by Upstream DNS Server", "Blocked", "Censored" or "Filtered" {{!RFC8914}},
   otherwise the EXTRA-TEXT field is discarded.

4. If the JSON object contains an "s" field and the sub-error code
   is not defined as applicable to the accompanying Extended DNS Error
   (EDE) code, the client MUST ignore the value of the "s" field
   and continue processing the remaining fields in accordance with this
   specification.

5. If the EXTRA-TEXT field does not contain at least one of the JSON
   names "c", "j", or "s", or if all of the fields that are present have
   empty values, the entire JSON object MUST be discarded.

6. If the JSON object contains a "c" field any of its Contact URIs
   with schemes not registered in the {{IANA-Contact}} registry are
   ignored. Remaining Contact URIs using registered schemes can be
   processed.

7. If the identity of the DNS server cannot be verified (e.g., when
   using opportunistic privacy such as {{Section 5 of !RFC8310}} or opportunistic discovery {{?RFC9462}}), the DNS client MUST ignore the "c", "j", and "o" fields, as these fields may influence end user behavior and are vulnerable to active attacks in the absence of resolver authentication. If the DNS response was received over an encrypted connection without server authentication, the client MAY process the "s" field and other parts of the response, as the "s" field is a registry-defined, enumerated value and does not contain free-form text.

8. If the DNS client uses an authenticated connection to the DNS server (e.g., when
   using a strict privacy profile for DNS-over-TLS ({{Section 5 of !RFC8310}}) or an authenticated DNS-over-HTTPS or DNS-over-QUIC connection), this mitigates both passive eavesdropping and client redirection (at the expense of providing no DNS service if such a connection is not available). In such cases, the DNS client MAY process the EXTRA-TEXT field of the DNS response.

9. The DNS client MUST ignore any other JSON names that it does not support.

> Note that the strict and opportunistic privacy profiles as defined in {{!RFC8310}} only apply to DoT; there has been
no such distinction made for DoH.

## Structured DNS Error (SDE) EDNS(0) Option Format {#SDE}

The Structured DNS Error (SDE) EDNS(0) option is used by a client to
indicate support for I-JSON encoding in the EXTRA-TEXT field of an
Extended DNS Error (EDE) option.

The SDE option has no OPTION-DATA. The OPTION-LENGTH field MUST
be set to 0. A server receiving an SDE option with a non-zero
OPTION-LENGTH MUST ignore the option.

The presence of the SDE option in a query indicates that the client
supports processing the EXTRA-TEXT field in accordance with this
specification.

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

  * Meaning: Network Operator Policy. The code indicates that the request was filtered according to a policy imposed by the operator of the local network (where local network is a relative term, e.g., it may refer to a Local Area Network or to the network of the ISP selected by the end user).

  * Applicability: Blocked

  * Reference: This-Document

  * Change Controller: IETF


## DNS Operator Policy {#policy-dns}

  * Number: 6

  * Meaning: DNS Operator Policy. The code indicates that the request was filtered according to policy determined by the operator of the DNS server. This is different from the "Network Operator Policy" code when a third-party DNS resolver is used.

  * Applicability: Blocked

  * Reference:  This-Document

  * Change Controller: IETF

# New Extended DNS Errors

This document defines an addition to the EDE codes defined in {{RFC8914}}.

## Extended DNS Error Code TBA1 - Blocked by Upstream DNS Server

The DNS server is unable to respond to the request
because the domain is on a blocklist due to an internal security policy
imposed by an upstream DNS server. This error code
is useful in deployments where a network-provided DNS forwarder
is configured to use an external resolver that filters malicious
domains. When the DNS forwarder receives a Blocked (15) error code
from the upstream DNS server, it can replace it with
"Blocked by Upstream DNS Server" (TBA1) before forwarding
the reply to the DNS client. Additionally, the EXTRA-TEXT field may
be forwarded to the DNS client.

Implementations should ensure that the communication channel with the
upstream DNS server provides adequate integrity protection to mitigate
the threats described in step 1 of {{client-processing}}.

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
  "o": "example.net Filtering Service",
  "l": "en"
}
~~~~~
{: #example-json title="JSON Returned in EXTRA-TEXT Field of Extended DNS Error Response"}

In {{example-json-minified}} the same content is shown with minified JSON (no
whitespace, no blank lines) with ```'\'``` line wrapping per {{?RFC8792}}.

~~~~~
{ "c":["tel:+358-555-1234567","sips:bob@bobphone.example.com"],\
"j":"malware present for 23 days",\
"s":1,\
"o":"example.net Filtering Service",\
"l":"en" }
~~~~~
{: #example-json-minified title="Minified Response"}

# Operational Considerations

When a forwarder receives an EDE option, whether or not (and how) to pass along JSON information in the
EXTRA-TEXT field to its client is implementation-dependent {{?RFC5625}} and depends on operator policy. Implementations MAY choose not to
forward the JSON information, or they MAY choose to create a new EDE option that conveys the information in the
"c", "s", and "j" fields encoded in the JSON object.

The application that triggered the DNS request may have a client security policy to override the contact information (e.g., redirect all complaint calls to a single contact point). In such cases, the content of the "c" attribute MAY be ignored.

## Backward Compatibility

Future extensions MUST NOT introduce mandatory JSON attributes, as existing implementations are required to ignore unknown JSON names (see {{client-processing}}).

# Security Considerations {#security}

## Authentication and Confidentiality

Security considerations in {{Section 6 of !RFC8914}} apply to this
document. {{!RFC8914}} cautions against relying on EDE information because it may be unauthenticated and transmitted in cleartext. This specification assumes the use of authenticated, integrity-protected DNS transports (e.g., DNS-over-TLS, DNS-over-HTTPS, or DNS-over-QUIC). Such transports MUST be based on TLS 1.3 {{!RFC8446}} or later.  Under these conditions, EDE information is integrity-protected, reducing the risks associated with relying on structured EDE content.

To minimize impact of active on-path attacks on the DNS channel, the
client validates the response as described in {{client-processing}}.

## Restrictions on Display of "c", "o", and "j" Fields

A client might choose to display the information in the "c" field
to the end user if and only if the encrypted resolver has sufficient
reputation, according to some client security policy (e.g.,
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
sub-error description for the "s" field to the end user.

The same client security policy considerations apply to the display of the "j" field, as it
contains free-form, human-readable text that may influence end user behavior.

When displaying the free-form text of "o", the client MUST
NOT make any of those elements into actionable (clickable) links and these
fields need to be rendered as text, not as HTML. The contact details of "c" can be made
into clickable links to provide a convenient way for end users to initiate, e.g., voice calls. The client might
choose to display the contact details only when the identity of the DNS server is verified.

Clients MUST NOT automatically initiate connections to URIs derived from the EXTRA-TEXT field. Doing so could allow a resolver to silently report client activity to third parties, enable denial-of-service reflection attacks, or be used to entrap a client. The restriction of Contact URI schemes to "sips", "tel", and "mailto" is intentional, as these schemes do not result in automatic HTTP connections.

Further, clients MUST NOT display the value of the `"o"` field to the end user unless one of the following
conditions is met:

  * The value matches a registered organization name listed in the {{IANA-Enterprise}} OR
  * The value consists solely of an organization name and does not contain any additional free-form content such
    as instructions, URLs, or messaging intended to influence end user behavior, as determined by client security policy or heuristics.

If the organization name cannot be verified through registry checks or heuristics, the client MUST NOT display the "o" field to the end user.

DNS clients MAY keep all fields conveyed in the EXTRA-TEXT field for evaluation according to the client security  policy. Such data MUST NOT be automatically trusted, displayed to end users, or used to influence security decisions without appropriate validation.

## Security Risks from Legacy DNS Forwarders

An attacker might inject (or modify) the EDE EXTRA-TEXT field with a
DNS proxy or DNS forwarder that is unaware of EDE. Such a DNS proxy or
DNS forwarder will forward that attacker-controlled EDE option.  To
prevent such an attack, clients can be configured to process EDE from
explicitly configured DNS servers or utilize RESINFO
{{?RFC9606}}.

# IANA Considerations {#IANA}

This document requests five IANA actions as described in the following subsections.

> Note to the RFC Editor: Please replace RFCXXXX with the RFC number assigned to this document and "TBA1" with the value assigned by IANA.

## Structured DNS Error EDNS Option

IANA is requested to register the following new EDNS(0) Option Code in the
"DNS EDNS0 Option Codes (OPT)" registry under the "Domain Name System (DNS) Parameters" registry group {{IANA-DNS}}:

Value:
: TBD

Name:
: Structured DNS Error

Status:
: Standard

Reference:
: RFC XXXX


##  New Registry for JSON Names {#IANA-Names}

This document requests IANA to create a new registry, entitled "EXTRA-TEXT JSON Names"
under "Extended DNS Error Codes" registry, which is under the "Domain Name System (DNS) Parameters" registry group {{IANA-DNS}}. The registration request for a new JSON name must include the
following fields:

JSON Name:
: Specifies the name of an attribute that is present in the JSON data enclosed in EXTRA-TEXT field. The name must follow the guidelines in {{name-spec}}.

Field Meaning:
: Provides a brief, human-readable label summarizing the purpose of the JSON attribute.

Short description:
: Includes a short description of the requested JSON name.

Mandatory (Y/N?):
: Indicates whether this attribute is mandatory or optional.

Specification:
: Provides a pointer to the reference document that specifies the attribute.

The registry is initially populated with the following values:

| JSON Name | Field Meaning  | Description                      |  Specification |
|:---------:|:---------------|:---------------------------------|:------------------:|
| c | contact| The contact details of the IT/InfoSec team to report misclassified DNS filtering | {{name-spec}} of RFCXXXX |
| j | justification | UTF-8-encoded {{!RFC5198}} textual justification for a particular DNS filtering | {{name-spec}} of RFCXXXX |
| s | sub-error | Integer representing the sub-error code for this DNS filtering case | {{name-spec}} of RFCXXXX |
| o | organization | UTF-8-encoded human-friendly name of the organization that filtered this particular DNS query | {{name-spec}} of RFCXXXX |
| l | language     | Indicates the language of the "j" and "o" fields as defined in {{!RFC5646}} | {{name-spec}} of RFCXXXX |
{: #reg-names title='Initial JSON Names Registry'}

New JSON names are registered via IETF Review ({{Section 4.8 of !RFC8126}}) and their formatting
constraints are described in {{name-spec}}.

## New Registry for Contact URI Scheme {#IANA-Contact}

This document requests IANA to create a new registry, entitled "Contact URI Schemes"
under "Extended DNS Error Codes"
registry, which is under the "Domain Name System (DNS) Parameters" registry group {{IANA-DNS}}. The registration request for a new Contact URI scheme has to include the
following fields:

* Name: URI scheme name.

* Meaning: Provides a short description of the scheme.

* Reference: Provides a pointer to an IETF-approved specification that defines
  the URI scheme.

The Contact URI scheme registry is initially populated with the
following schemes:

| Name      | Meaning           | Reference     |
|:---------:|:------------------|:-------------:|
| sips      | SIP Call           | {{!RFC5630}} |
| tel       | Telephone Number   | {{!RFC3966}} |
| mailto    | Internet mail      | {{!RFC6068}} |

{: #reg-contact='Initial Contact URI Schemes Registry'}

The registration procedure for adding new Contact URI schemes to the "Contact URI Schemes" registry is "IETF
Review" as defined in {{Section 4.8 of !RFC8126}}.

## New Registry for DNS Sub-Error Codes {#IANA-SubError}

This document requests IANA to create a new registry, entitled "Sub-Error Codes"
under "Extended DNS Error Codes" registry, which is under the "Domain Name System (DNS) Parameters" registry group {{IANA-DNS}}. The registration request for a new sub-error code must include the
following fields:

* Number: Is the wire format sub-error code (range 0-255).

* Meaning: Provides a short description of the sub-error.

* EDE Codes Applicability: Indicates which Extended DNS Error (EDE) Codes apply to this sub-error code.

* Reference: Provides a pointer to an IETF-approved specification that registered
  the code and/or an authoritative specification that describes the
  meaning of this code.

The Sub-Error Code registry is initially populated with the
following values:

| Number | Meaning | EDE Codes Applicability | Reference |
|:------:|:--------|:------------------------|:----------|
| 0 | Reserved| Not used | {{policy-reserved}} of this document |
| 1 | Malware | "Blocked", "Blocked by Upstream DNS Server", "Filtered" | Section 5.5 of {{!RFC5901}} |
| 2 | Phishing | "Blocked", "Blocked by Upstream DNS Server", "Filtered" | Section 5.5 of {{!RFC5901}} |
| 3 | Spam | "Blocked", "Blocked by Upstream DNS Server", "Filtered" | Page 289 of {{!RFC4949}} |
| 4 | Spyware | "Blocked", "Blocked by Upstream DNS Server", "Filtered" | Page 291 of {{!RFC4949}} |
| 5 | Network operator policy | "Blocked" | {{policy-network}} of this document |
| 6 | DNS operator policy | "Blocked" | {{policy-dns}} of this document |
{: #reg title='Initial Sub-Error Code Registry'}

The registration procedure to add New Sub-Error Codes is IETF Review as defined in {{Section 4.8 of !RFC8126}}.

## New Extended DNS Error Code

IANA is requested to assign the following Extended DNS Error code from the "Extended DNS Error Codes"
registry under the "Domain Name System (DNS) Parameters" registry group {{IANA-DNS}}:

| INFO-CODE | Purpose                          | Reference |
|:---------:|:---------------------------------|:---------:|
| TBA1      | Blocked by Upstream DNS Server   | RFCXXXX   |
{: #reg-ede title='New DNS Error Code'}

--- back

# Interoperation with RPZ Servers

This appendix provides a non-normative guidance for operation with a Response Policy Zones (RPZ) server {{RPZ}} that
indicates filtering with a NXDOMAIN response with the Recursion
Available bit cleared (RA=0). This guidance is provided to ease interoperation with RPZ.

When a DNS client supports this specification, it includes the
SDE option in its DNS query.

If the server does not support this specification and is performing
RPZ filtering, the server ignores the SDE option in the DNS query and
replies with NXDOMAIN and RA=0. The DNS client can continue to accept
such responses.

If the server does support this specification and is performing RPZ
filtering, the server can use the SDE option in the query to identify
an SDE-aware client and respond appropriately (that is, by generating
a response described in {{server-response}}) as NXDOMAIN and RA=0
are not necessary when generating a response to such a client.

# Implementation Status

> Note to the RFC Editor: please remove this appendix prior publication.

At IETF#116, Gianpaolo Scalone (Vodafone) and Ralf Weber (Akamai) presented an implementation of this specification. More details can be found at {{Impl-1}}.



# Acknowledgements
{:numbered="false"}

Thanks to Vittorio Bertola, Wes Hardaker, Ben Schwartz, Erid Orth,
Viktor Dukhovni, Warren Kumari, Paul Wouters, John Levine, Bob
Harold, Mukund Sivaraman, Gianpaolo Angelo Scalone, Mark Nottingham, Stephane Bortzmeyer, Daniel Migault, and Stephen Farrell for the comments.

Thanks to Ralf Weber and Gianpaolo Scalone for sharing details about their implementation.

Thanks Petr Špaček, Di Ma and Matt Brown for the DNS directorate reviews, and Joseph Salowey for the Security directorate review.

Thanks Paul Kyzivat for the Art review.

Thanks to Éric Vyncke for the AD review.



