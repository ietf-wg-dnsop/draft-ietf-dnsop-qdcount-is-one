---
title: In the DNS, QDCOUNT is (usually) One
docname: draft-ietf-dnsop-qdcount-is-one-04
updates: 1035

submissiontype: IETF
ipr: trust200902
area: Internet
wg: DNSOP Working Group
kw: Internet-Draft
cat: std

coding: us-ascii
pi:
  - toc
  - symrefs
  - sortrefs

author:
  -
    ins: R. Bellis
    name: Ray Bellis
    org: Internet Systems Consortium, Inc.
    abbrev: ISC
    street: PO Box 360
    city: Newmarket
    code: NH 03857
    country: US
    phone: +1 650 423 1300
    email: ray@isc.org
  -
    ins: J. Abley
    name: Joe Abley
    org: Cloudflare
    city: Amsterdam
    country: NL
    phone: +31 6 45 56 36 34
    email: jabley@cloudflare.com

--- abstract

This document updates RFC 1035 by constraining the allowed value of the
QDCOUNT parameter in DNS messages with OPCODE = 0 (QUERY) to a maximum
of one, and specifies the required behaviour when values that are not
allowed are encountered.

--- middle

# Introduction

The DNS protocol {{!RFC1034}}{{!RFC1035}} includes a parameter
QDCOUNT in the DNS message header, whose value is specified to mean
the number of questions in the Question Section of a DNS message.

In a general sense it seems perfectly plausible for the QDCOUNT
parameter, an unsigned 16-bit value, to take a considerable range
of values. However, in the specific case of messages that encode
DNS queries and responses (messages with OPCODE = 0) there are other
limitations inherent in the protocol that constrain values of QDCOUNT
to be either 0 or 1. In particular, several parameters specified
for DNS response messages such as AA and RCODE have no defined
meaning when the message contains multiple queries, since there is
no way to signal which question those parameters relate to.

In this document we briefly survey the existing written DNS
specification; we provide a description of the semantic and practical
requirements for DNS queries that naturally constrain the allowable
values of QDCOUNT; and we update the DNS base specification to
clarify the allowable values of the QDCODE parameter in the specific
case of DNS messages with OPCODE = 0.

# Terminology used in this document

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY",
and "OPTIONAL" in this document are to be interpreted as described
in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear
in all capitals, as shown here.

# QDCOUNT is (usually) One

A brief summary of the guidance provided in the existing DNS
specification ({{!RFC1035}} and many other documents) for the use
of QDCOUNT can be found in {{Survey}}.  While the specification is
clear in many cases, in the specific case of OPCODE = 0 there is
some ambiguity which this document aims to eliminate.

# Updates to RFC 1035

A DNS message with OPCODE = 0 MUST NOT include a QDCOUNT parameter
whose value is greater than 1. It follows that the Question Section
of a DNS message with OPCODE = 0 MUST NOT contain more than one
question.

A DNS message with OPCODE = 0 and QDCOUNT > 1 MUST be treated as
an incorrectly-formatted message.  The value of the RCODE parameter
in the response message MUST be set to 1 (FORMERR).

Middleboxes (e.g. firewalls) that process DNS messages in order
to eliminate unwanted traffic SHOULD treat messages with OPCODE =
0 and QDCOUNT > 1 as malformed traffic and return a FORMERR response
as described above.  Such firewalls MUST NOT treat messages with
OPCODE = 0 and QDCOUNT = 0 as malformed.  See Section 4 of {{?RFC8906}}
for further guidance.

# Security Considerations

This document clarifies the DNS specification and aims to improve
interoperability between different DNS implementations. In general,
the elimination of ambiguity seems well-aligned with security
hygiene.

# IANA Considerations

This document has no IANA actions.

# Acknowledgements

The clarifications in this document were prompted by questions posed
by Ted Lemon, which reminded the authors of earlier, similar questions
and motivated them to pick up their pens. Ondrej Sury, Warren Kumari,
Peter Thomassen, Mark Andrews, Lars-Johan Liman, Jim Reid and Niall
O'Reilly provided useful feedback.

--- back

# Guidance for the use of QDCOUNT in the DNS Specification {#Survey}

The DNS Specification provides some guidance about the values of
QDCOUNT that are appropriate in various situations. A brief summary
of this guidance is collated below.

## OPCODE = 0 (QUERY) and 1 (IQUERY)

{{!RFC1035}} significantly predates the use of the normative
requirements keywords specified in BCP 14 {{!RFC2119}} {{!RFC8174}},
and parts of it are consequently somewhat open to interpretation.

Section 4.1.2 ("Question section format") has this to say about
QDCOUNT:

> The section contains QDCOUNT (usually 1) entries

The only documented exceptions within {{!RFC1035}} relate to the IQuery
Opcode, where the request has "an empty question section" (QDCOUNT = 0),
and the response has "zero, one, or multiple domain names for the
specified resource as QNAMEs in the question section". The IQuery OpCode
was made obsolete in {{!RFC3425}}.

In the absence of clearly expressed normative requirements, we rely
on other text in {{!RFC1035}} that makes use of the definite article
or other text that implies a singular question and, by implication,
QDCOUNT = 1.

For example, Section 4.1:

> the question for the name server

and:

> The question section contains fields that describe a question to a
> name server

and in Section 4.1.1. ("Header section format"):

> AA Authoritative Answer - this bit is valid in responses,
>    and specifies that the responding name server is an
>    authority for the domain name in question section.

DNS Cookies {{?RFC7873}} in Section 5.4 allow a client to receive
a valid Server Cookie without sending a specific question by sending
a request (QR = 0) with OPCODE = 0 and QDCOUNT = 0, with the resulting
response also containing no question.

DNS Zone Transfer Protocol (AXFR) {{?RFC5936}} in Section 2.2 allows
an authoritative server optionally to send a response message
(QR = 1) to a standard AXFR query (OPCODE = 0, QTYPE=252) with
QDCOUNT = 0 in the second or subsequent message of a multi-message
response.

## OPCODE = 4 (NOTIFY)

DNS Notify {{?RFC1996}} also lacks a clearly defined range of values
for QDCOUNT.  Section 3.7 says:

> A NOTIFY request has QDCOUNT > 0

but all other text in the RFC talks about the <QNAME, QCLASS, QTYPE>
tuple in the singular.

## OPCODE = 5 (UPDATE)

DNS Update {{?RFC2136}} renames the QDCOUNT field to ZOCOUNT, but
the value is constrained to be one by Section 2.3 ("Zone Section"):

> All records to be updated must be in the same zone, and therefore the
> Zone Section is allowed to contain exactly one record.

## OPCODE = 6 (DNS Stateful Operations, DSO)

DNS Stateful Operations {{?RFC8490}} (DSO - OpCode 6) attempts to
preserve compatibility with the standard DNS 12 octet header, and
does so by requiring that all four of the section count values be
set to zero.

## Conclusion

There is no text in {{!RFC1035}} that describes how other parameters
in the DNS message such as AA, RCODE should be interpreted in the
case where a message includes more than one question. An originator
of a query with QDCOUNT > 1 can have no expectations of how it will
be processed, and the receiver of a response with QDCOUNT > 1 has
no guidance for how it should be interpreted.

The allowable values of QDCOUNT seem to be clearly specified for
OPCODE = 4 (NOTIFY), OPCODE = 5 (UPDATE) and OPCODE = 6 (DNS Stateful
Operations, DSO). OPCODE = 1 (IQUERY) is obsolete and OPCODE = 2
(STATUS) is not specified. OPCODE = 3 is reserved.

However, the allowable values of QDCOUNT for OPCODE = 0 (QUERY) are
specified in {{?RFC1035}} without the clarity of normative language,
and this looseness of language results in some ambiguity.

