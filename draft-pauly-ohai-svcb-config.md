---
title: "Distribution of Oblivious Configurations via Service Binding Records"
abbrev: "Oblivious Configs in SVCB"
category: info

docname: draft-pauly-ohai-svcb-config-latest
ipr: trust200902
area: "Security"
workgroup: "Oblivious HTTP Application Intermediation"
keyword: Internet-Draft
venue:
  group: "Oblivious HTTP Application Intermediation"
  type: "Working Group"
  mail: "ohai@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/ohai/"

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
 -
    name: Tommy Pauly
    organization: Apple Inc.
    email: tpauly@apple.com

normative:

informative:


--- abstract

This document defines a parameter that can be included in SVCB and HTTPS
DNS resource records to denote that a service is accessible as an Oblivious
HTTP target, along with one or more oblivious key configurations.

--- middle

# Introduction

Oblivious HTTP {{!OHTTP=I-D.draft-ietf-ohai-ohttp}} allows clients to encrypt
messages exchanged with an HTTP server accessed via a proxy, in such a way
that the proxy cannot inspect the contents of the message and the target HTTP
server does not discover the client's identity. In order to use Oblivious
HTTP, clients need to possess a key configuration to use to encrypt messages
to the oblivious target.

Since Oblivious HTTP deployments will often involve very specific coordination
between clients, proxies, and targets, the key configuration can often be
shared in a bespoke fashion. However, some deployments involve clients
discovering oblivious targets more dynamically. For example, a network may
want to advertise a DNS resolver that is accessible over Oblivious HTTP
and applies local network resolution policies. Clients can work with
trusted proxies to access these target servers.

This document defines a mechanism to distribute Oblivious HTTP key
configurations in DNS records, as a parameter that can be included in SVCB and
HTTPS DNS resource records {{!SVCB=I-D.draft-ietf-dnsop-svcb-https}}.
The presence of this parameter indicates that a service is an oblivious
target.

This mechanism does not aid in the discovery of proxies to use to access
oblivious targets; the configurations of proxies is out of scope for this
document.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# The oblivious-keys SvcParamKey

The "oblivious-keys" SvcParamKey {{iana}} is used to convey one or more
key configurations that can be used by clients to issue oblivious requests
to a target server described by the SVCB record.

In wire format, the value of the parameter is one or more `KeyConfig`
structures {{OHTTP}} concatenated together. In presentation format,
the value is the same concatenated `KeyConfig` structures encoded
in Base64 {{!BASE64=RFC4648}}.

The meaning of the "oblivious-keys" parameter depends on the scheme
of the SVCB record. This document defines the interpretation for
the "https" {{SVCB}} and "dns" {{!DNS-SVCB=I-D.draft-ietf-add-svcb-dns}}
schemes. Other schemes that want to use this parameter MUST define the
interpretation and meaning of the key.

## Use in HTTPS service records

For the "https" scheme, which uses the HTTPS RR type instead of SVCB,
the presence of the "oblivious-keys" parameter means that the service
being described is an Oblivious HTTP service that uses the default
"message/bhttp request" media type {{OHTTP}}
{{!BINARY-HTTP=I-D.draft-ietf-httpbis-binary-message}}.

When present in an HTTPS record, the "oblivious-keys" MUST be included
in the mandatory parameter list, to ensure that implementations that
do not understand the key do not interpret this service as a generic
HTTP service.

Clients MUST validate that they can parse the value of "oblivious-keys"
as a valid key configuration before attempting to use the service.

## Use in DNS server SVCB records

For the "dns" scheme, as defined in {{DNS-SVCB}}, the presence of
the "oblivious-keys" parameter means that the DNS server being
described is an Oblivious DNS over HTTP (DoH) service. The default
media type expected for use in Oblivious HTTP to DNS resolvers
is "application/dns-message" {{!DOH=RFC8484}}.

The "oblivious-keys" parameter is only defined for use with DoH, so
the "alpn" SvcParamKey MUST indicate support for a version of HTTP
and the "dohpath" SvcParamKey MUST be present. The "oblivious-keys"
MUST also be included in the mandatory parameter list, to ensure
that implementations that do not understand the key do not interpret
this service as a generic DoH service.

Clients MUST validate that they can parse the value of "oblivious-keys"
as a valid key configuration before attempting to use the service.

### Handling Oblivious DoH Configurations

Oblivious DoH was originally defined in
{{?ODOH=I-D.draft-pauly-dprive-oblivious-doh}}. This version of
Oblivious DoH uses a different key configuration format than
generic Oblivious HTTP. SVCB records using the "dns" scheme
MAY include an ObliviousDoHConfigs structure (including the
redundant length field) in place of the concatenated KeyConfigs
structure, since two structures are mutually exclusive.

# Deployment Considerations

Deployments that add the "oblivious-keys" SvcParamKey need to be
careful to add this only to services meant to be accessed using
Oblivious HTTP. Information in a single SVCB record that contains
"oblivious-keys" only applies to the oblivious service, not
other HTTP services.

If a service offers both traditional HTTP and oblivious HTTP, these can
be represented by separate SVCB or HTTPS records, both with and
without the "oblivious-keys" SvcParamKey.

# Security and Privacy Considerations

As discussed in {{OHTTP}}, client requests using Oblivious HTTP
can only be linked by recognizing the key configuration. In order to
prevent unwanted linkability and tracking, clients using any key
configuration discovery mechanism need to be concerned with attacks
that target a specific user or population with a unique key configuration.

There are several approaches clients can use to mitigate key targetting
attacks. Clients SHOULD employ some technique to mitigate this attack.
Possible mitigations include:

- Validating that SVCB or HTTPS records including the "oblivious-keys"
are protected by DNSSEC {{?DNSSEC=RFC4033}}. This prevents attacks
where a unique response is generated for each client of a resolver.

- Coordination with the oblivious proxy to recognize anomalous key
configurations. Trusted proxies can inspect the truncated key ID on
oblivious requests to a target (and clients could choose to additionally
share entire key configurations with proxies), allowing the proxy to
reject requests if they detect that one user or a set of users is
receiving a different key configuration than others.

- Clients can retrieve the SVCB or HTTPS records through multiple resolvers,
or in a way that doesn't reveal any client information, to make it difficult
to target a user.

- Clients can check keys with other clients or against public logs to
validate that the keys they receive are not unique.

# IANA Considerations {#iana}

IANA is requested to add the following entry to the SVCB Service Parameters
registry ({{SVCB}}).

| Number  | Name           | Meaning                      | Reference       |
| ------- | -------------- | ---------------------------- | --------------- |
| TBD     | oblivious-keys | Oblivious HTTP keys          | (This document) |

--- back
