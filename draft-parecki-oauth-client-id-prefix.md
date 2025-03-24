---
title: "OAuth 2.0 Client ID Prefix"
abbrev: "Client ID Prefix"
category: info

docname: draft-parecki-oauth-client-id-prefix-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Web Authorization Protocol"
keyword:
 - oauth
venue:
  group: "Web Authorization Protocol"
  type: "Working Group"
  mail: "oauth@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/oauth/"
  github: "aaronpk/oauth-client-id-prefix"
  latest: "https://drafts.aaronpk.com/oauth-client-id-prefix/draft-parecki-oauth-client-id-prefix.html"

author:
  -
    fullname: "Aaron Parecki"
    organization: Okta
    email: "aaron@parecki.com"
  -
    fullname: "Daniel Fett"
    organization: Authlete
    email: "mail@danielfett.de"
  -
    fullname: "Joseph Heenan"
    organization: Authlete
    email: "joseph@heenan.me.uk"

normative:
  RFC6749:
  RFC5280:
  RFC7515:
  RFC8414:
  DID-Core:
    title: "DID Core"
    target: https://www.w3.org/TR/did-core/
    date: 2022-07-19
  OpenID.Federation:
    title: "OpenID Federation 1.0"
    date: 2024-05-17
    target: https://openid.net/specs/openid-federation-1_0.html
    author:
      - name: R. Hedberg
        org: independent
      - name: M.B. Jones
        org: Self-Issued Consulting
      - name: A.Å. Solberg
        org: Sikt
      - name: J. Bradley
        org: Yubico
      - name: G. De Marco
        org: independent
      - name: V. Dzhuvinov
        org: Connect2id

informative:
  RFC7591:
  RFC9068:
  I-D.draft-parecki-oauth-client-id-metadata-document:
  OpenID:
    title: "OpenID Connect Core 1.0"
    date: 2023-12-15
    target: https://openid.net/specs/openid-connect-core-1_0.html
    author:
      - name: N. Sakimura
        org: NAT.Consulting
      - name: J. Bradley
        org: Yubico
      - name: M. Jones
        org: Self-Issued Consulting
      - name: B. de Medeiros
        org: Google
      - name: C. Mortimore
        org: Disney


--- abstract

This specification defines the concept of a Client Identifier Prefix to enable Authorization Servers and Clients to use more than one mechanism to obtain and validate Client metadata.

--- middle

# Introduction

A Client Identifier is used by an OAuth 2.0 Client to identify itself to an Authorization Server. The Client Identifier is used in the Authorization Request and various other places throughout OAuth flows. In ecosystems where more than one method of obtaining and validating Client metadata is used, it is necessary to indicate unambiguously which method is used. This specification defines a structure for Client Identifiers that includes a prefix indicating the Client Identifier Prefix.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Client Identifier Prefix

This specification defines the concept of a Client Identifier Prefix that indicates how an Authorization Server is supposed to interpret the Client Identifier and associated data in the process of Client identification, authentication, and authorization. The Client Identifier Prefix enables deployments of this specification to use different mechanisms to obtain and validate metadata of the Client beyond the scope of {{RFC6749}}.

The Client Identifier Prefix is a string that MAY be communicated by the Client in a prefix within the `client_id` parameter in the Authorization Request. A fallback to pre-registered Clients as in {{RFC6749}} or a default Client Identifier Prefix is in place as a default mechanism in case no Client Identifier Prefix was provided. A certain Client Identifier Prefix may require the Client to sign the Authorization Request as means of authentication and/or pass additional parameters and require the Authorization Server to process them.

## Syntax

In the `client_id` Authorization Request parameter and other places where the Client Identifier is used, the Client Identifier Prefixes are prefixed to the usual Client Identifier, separated by a `:` (colon) character:

    <client_id_prefix>:<orig_client_id>

Here, `<client_id_prefix>` is the Client Identifier Prefix and `<orig_client_id>` is an identifier for the Client within the namespace of that prefix. See {{client_identifier_prefixes}} for Client Identifier Prefixes defined by this specification.

Authorization Servers MUST use the presence of a `:` (colon) character and the content preceding it to determine whether a Client Identifier Prefix is used. If a `:` character is present, and the content preceding it is a recognized and supported Client Identifier Prefix value, the Authorization Server MUST interpret the Client Identifier according to the given Client Identifier Prefix. The Client Identifier Prefix is defined as the string before the (first) `:` character. If the Authorization Server does not support the Client Identifier Prefix, the Authorization Server MUST refuse the request.

For example, an Authorization Request might contain `client_id=client_attestation:example-client` to indicate that the `client_attestation` Client Identifier Prefix is to be used and that within this prefix, the Client can be identified by the string `example-client`.

Note that the Client may need to determine which Client Identifier Prefixes the Authorization Server supports prior to sending the Authorization Request in order to ensure the client's preferred prefix is supported.


## Fallback for Unrecognized Client ID Prefixes

If a `:` character is not present in the Client Identifier, the Authorization Server MUST treat the Client Identifier as referencing a pre-registered client. This is equivalent to the {{RFC6749}} default behavior, i.e., the Client Identifier needs to be known to the Authorization Server in advance of the Authorization Request. The Client metadata is pre-registered using {{RFC7591}} or through out-of-band mechanisms.

For example, if an Authorization Request contains `client_id=example-client`, the Authorization Server would interpret the Client Identifier as referring to a pre-registered client.

If a `:` character is present in the Client Identifier but the value preceding it is not a recognized and supported Client Identifier Prefix value, the Authorization Server MAY treat the Client Identifier as having a default Client Identifier Prefix.

For example, an Authorization Request containing a `client_id` value of `https://client.example.com/metadata.json` could be interpreted by the Authorization Server as referring to a Client ID Metadata Document {{I-D.draft-parecki-oauth-client-id-metadata-document}}, with the default Client Identifier Prefix being `client-id-metadata-document`.

From this definition, it follows that pre-registered clients MUST NOT contain a `:` character preceded immediately by a supported Client Identifier Prefix value in the first part of their Client Identifier.


### Example

Deployments that use `https` URLs as client IDs and that have only one way to resolve client metadata from the URL, MAY use only the full https URL as the client ID. If there is only one way to resolve client metadata then there is no ambiguity in which metadata retrieval method to use, and are not susceptible to client identifier mixup attacks as described in {{client-id-mixups}}.

For example, an authorization server using only the Client ID Metadata Document {{I-D.draft-parecki-oauth-client-id-metadata-document}} method to retrieve client metadata MAY accept client IDs such as:

    https://client.example.com/metadata.json

This results in this non-normative example of an authorization request:

    GET /authorize?
      response_type=code
      &client_id=https%3A%2F%2Fclient.example.org%2Fmetadata.json
      &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcallback
      &code_challenge=GdE4nqBrwRxQfN2Y8fq3rrYk_kkpwg6tQ74J94-2nHw
      &code_challenge_method=S256
      &scope=write


## Defined Client Identifier Prefixes {#client_identifier_prefixes}

This specification defines the following Client Identifier Prefixes, followed by the examples where applicable:

* `redirect_uri`: This value indicates that the Client Identifier (without the prefix `redirect_uri:`) is the Client's Redirect URI (or Response URI when Response Mode `direct_post` is used). The Authorization Request MUST NOT be signed. The Client MAY omit the `redirect_uri` Authorization Request parameter. Example Client Identifier: `redirect_uri:https%3A%2F%2Fclient.example.org%2Fcb`.

* `client_id_metadata_document`: This value indicates that the Client Identifer (without the prefix `client_id_metadata_document:`) is the client's Client ID Metadata Document {{I-D.draft-parecki-oauth-client-id-metadata-document}}.

* `https`: This Client Identifier Prefix MUST NOT be registered.


# Example

The following is a non-normative example of an authorization request with the `client_id_metadata_document` Client ID Prefix:

    GET /authorize?
      response_type=code
      &client_id=client_id_metadata_document:https%3A%2F%2Fclient.example.org%2Fmetadata.json
      &redirect_uri=https%3A%2F%2Fclient.example.org%2Fredirect
      &code_challenge=GdE4nqBrwRxQfN2Y8fq3rrYk_kkpwg6tQ74J94-2nHw
      &code_challenge_method=S256
      &scope=write

# Authorization Server Metadata {#as-metadata}

Authorization servers that publish Authorization Server Metadata ({{RFC8414}}) MUST include the following properties to indicate support for client ID prefixes as described in this specification.

`client_id_prefixes_supported`:
: REQUIRED. A JSON array of strings indicating the clients ID prefixes supported by this authorization server.


# Security Considerations

## Client Identifier Mixups {#client-id-mixups}

Confusing Clients using a Client Identifier Prefix with those using none can lead to various mixup attacks. Therefore, Authorization Servers MUST always use the full Client Identifier, including the prefix if provided, within the context of the Authorization Server or its responses to identify the client. This refers in particular to places where the Client Identifier is used in {{RFC6749}} as well as in any artifacts such as the `aud` claim of JWT access tokens {{RFC9068}}.



# IANA Considerations

## OAuth Authorization Server Metadata Registry

The following authorization server metadata value is defined by this specification and (TBD) registered in the IANA "OAuth Authorization Server Metadata" registry established in OAuth 2.0 Authorization Server Metadata {{RFC8414}}.

* Metadata Name: `client_id_prefixes_supported`:
* Metadata Description: A JSON array of strings indicating the client ID prefixes supported by the authorization server.
* Change Controller: IETF
* Specification Document: {{as-metadata}} of [[ this specification ]]


--- back

# Acknowledgments
{:numbered="false"}

The authors would like to thank the following people for their contributions and
reviews of this specification:

Brian Campbell, Emelia Smith.


