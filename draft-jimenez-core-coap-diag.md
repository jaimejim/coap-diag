---
v: 3

title: "CoAP Diagnostic Message Notation"
abbrev: CoAP Diag
docname: draft-jimenez-core-coap-diag-latest

category: info
stream: IETF
area: Internet
workgroup: CoRE Working Group

venue:
  mail: core@ietf.org
  github: jaimejim/coap-diag

author:
- name: Jaime Jimenez
  org: Ericsson
  email: jaime@iki.fi
- name: Carsten Bormann
  org: Universität Bremen TZI
  email: cabo@tzi.org

normative:
  RFC7252:
  RFC6690:
  RFC8132: etch
  RFC7641: observe
  RFC7959: block
  RFC9176:
  RFC8949:
  RFC3986: uri

informative:
  RFC9112: http11
  RFC9203:
  RFC9200:
  RFC8742:
  RFC8610: cddl
  I-D.ietf-core-comi:
  I-D.ietf-core-coap-pubsub:
  I-D.ietf-cbor-edn-literals:

entity:
  SELF: "[RFC-XXXX]"

--- abstract

This document defines a text notation for representing Constrained Application Protocol (CoAP) request/response exchanges in Internet-Drafts and RFCs. The notation balances human readability with mechanical validation: a reader can follow an exchange at a glance, and tooling can be built to parse the message structure and check each payload against its declared content format. The notation is application-layer by default, with message-layer detail added only where an example depends on it. It is a recommended convention for use in documents and with authoring tools such as kramdown-rfc, not a conformance target; it does not change CoAP or any on-the-wire encoding.

--- middle

# Introduction {#introduction}

The Constrained Application Protocol (CoAP) {{RFC7252}} defines a request/response model in which clients send requests on resources hosted by servers, receiving responses. Documents that specify or use CoAP illustrate these exchanges with examples: a request line, a set of options, and a payload, followed by the response. These examples are how a reader first understands a protocol interaction, and they are frequently the part of a document that implementers copy from.

Authors write these examples in divergent, ad hoc notations. Some split the request target into individual `Uri-Path` and `Uri-Query` option lines; others compose it into a single URI on the first line. Some prefix each message with `Header:` and each body with `Payload:`; others use `Req:` and `Res:`, or no markers at all. The same CBOR payload appears with and without a declared content format, and response codes are written both as `2.05 Content` and as `Header: Content (Code=2.05)`. {{survey}} surveys seven such variants drawn from published RFCs and active drafts.

This divergence has two costs. A reader who moves between documents must re-learn the convention each time. More importantly, no widely used notation is both readable and machine-checkable, so a malformed CBOR or JSON payload in an example is caught only by a human reviewer, if at all. Tooling that could validate payloads exists ({{tool-integration}}), but it needs a predictable structure to parse first.

This document defines a single notation for CoAP exchanges, for use in documents and in authoring tools. The notation follows HTTP message syntax {{RFC9112}} where practical, composes URIs on the first line, declares the content format of each payload, and represents payloads in the text notation appropriate to that format. It covers requests, responses, and their options and payloads. It does not define a rendering for transport mechanisms such as block-wise transfer {{RFC7959}}, multicast requests, or suppressed responses {{?RFC7967}}; a document that illustrates those uses whatever notation it finds suitable. It does not change CoAP {{RFC7252}}, and it does not define or alter any message encoding on the wire.

# Terminology {#terminology}

This document uses the following terms.

{:vspace}
request line:
: The first line of a request. It carries the text name for the CoAP method and the request target, for example `GET /temperature`.

response line:
: The first line of a response. It carries the response code in `X.YY` form and a reason phrase, for example `2.05 Content`.

option line:
: A line of the form `Name: value` that represents one CoAP option, using the registered option name.

annotation:
: Additional information added to a request, response, or option line.

Content-Format annotation:
: In an option line for the `Content-Format` (or `Accept`) option, the content-format number is followed by its media type (and parameters, if any) in parentheses, for example:

      Content-Format: 60 (application/cbor)

payload:
: The message body, separated from the option lines by a single blank line and written in the text notation of its content format.

transport annotation:
: An optional parenthetical on the request or response line that records message-layer detail such as the message type or token, for example `(CON, Token=0x64)`.

EDN:
: CBOR Diagnostic Notation, the text representation of CBOR data items originally defined in {{Section 8 of RFC8949}} and {{Appendix G of RFC8610}} and now in {{I-D.ietf-cbor-edn-literals}}.[^Namechange]

[^Namechange]: The name EDN and its expansion is currently under discussion in the CBOR WG and might change.

# Design Principles {#principles}

The notation follows five principles.

{:vspace}
Familiar structure:
: The notation follows HTTP message syntax {{RFC9112}} where practical, so that a reader accustomed to HTTP examples can read a CoAP exchange without a legend.

Concise but unambiguous:
: The notation omits redundant labels while keeping the structure parsable. The method or response code, the options, and the payload each occupy a distinct position.

Payload-format-aware:
: A payload is preceded by a Content-Format annotation that names its media type, so that a tool can dispatch the payload to the correct validator. The diagnostic payload of an error response is the exception, as described in {{payload}}.

Wire-faithful:
: Options are written with their registered CoAP names, and request targets are composed into URIs rather than invented. The notation reflects the protocol it illustrates.

Transport-agnostic by default:
: The notation describes the application layer. CoAP message-layer detail is added only where an example depends on it, using transport annotations ({{transport}}).

# Notation {#notation}

A request is written as a request line, zero or more option lines, and an optional payload ({{fig-request}}). A response has the same structure with a response line in place of the request line ({{fig-response}}).

~~~~
METHOD /path[?query]
[Option: value]
[Option: value]

[payload]
~~~~
{: #fig-request title="Request structure"}

~~~~
CODE Reason-Phrase
[Option: value]
[Option: value]

[payload]
~~~~
{: #fig-response title="Response structure"}

## Request Line {#request-line}

The request line carries the CoAP method followed by the request target.
The method is written as its (mostly) uppercase name, for example `GET`, `POST`, `PUT`, `DELETE`, `FETCH`, `PATCH`, or `iPATCH` {{RFC8132}}.
The target is a composed path shown as a URI reference usually beginning with `/` (i.e., in absolute-path reference form, see {{Section 4.2 of -uri}}).
Where the host and/or URI scheme is relevant, the full URI form is used, for example `POST coap://broker.example.com/ps`.

## Response Line {#response-line}

The response line carries the response code in `X.YY` form followed by its reason phrase, for example `2.01 Created` or `2.05 Content`. The reason phrase is the name registered for that code in the IANA registry established in {{Section 12.1.2 of RFC7252}}.

## Options {#options}

Each option occupies one line of the form `Name: value`, using the option name registered in the "CoAP Option Numbers" registry. Uri-Path segments are composed into the target on the request line rather than written as separate `Uri-Path` option lines, and Uri-Query parameters are composed after a `?`. Location options in a response are composed in the same way and represented as an option line, for example `Location-Path: /ps/h9392`.

## Content-Format Annotation {#content-format}

An option line for `Content-Format` or `Accept` carries the content-format number followed by the corresponding media type in parentheses:

~~~~
Content-Format: TBD606 (application/core-pubsub+cbor)
~~~~

The number is the value a sender places on the wire; the parenthetical names the media type for the reader and for validation tooling ({{tool-integration}}). Where a code point has not yet been assigned, a placeholder is used in its place, written as `TBDnnn` or `NNN`, and resolved once IANA assigns the value. The examples in this document use `TBD606` for the `application/core-pubsub+cbor` format defined in {{I-D.ietf-core-coap-pubsub}}, which is not yet registered.

## Payload {#payload}

A single blank line separates the last option line from the payload. The payload is written in the text notation of its content format:

{:vspace}
CBOR-based formats:
: EDN, with inline comments where they aid the reader. A CBOR sequence {{RFC8742}} is written as the comma-separated EDN form, as in Variant 6 of {{survey}}.

JSON-based formats:
: JSON.

Link-format:
: CoRE Link Format {{RFC6690}}.

Binary content beyond CBOR:
: A hexadecimal dump using the `h'...'` notation of EDN {{RFC8949}}.

The diagnostic payload of an error response ({{Section 5.5.2 of RFC7252}}) is written as a text string with no Content-Format annotation, as shown in {{fig-error}}.

A payload line that exceeds the available width is folded using the convention of {{?RFC8792}}, so that a tool reconstructs the original payload by removing the folds before parsing.

## Transport Annotations {#transport}

Where an example depends on CoAP message-layer behavior, the detail is added in parentheses on the request or response line. Common annotations record the message type (`CON`, `NON`, `ACK`), the token, and the message ID, as defined in {{RFC7252}}:

~~~~
GET /temperature (CON, Token=0x64)

2.05 Content (ACK)
Observe: 9
Max-Age: 15
Content-Format: 0 (text/plain)

22.3 Cel
~~~~
{: #fig-transport title="Exchange with transport annotations for message type and token"}

## Multiple Messages {#multiple}

An exchange of more than two messages is written as a sequence of requests and responses, each separated from the next by a blank line. A short label can precede a message where it clarifies the flow.

## Grammar {#grammar}

{{fig-grammar}} sketches the structure in ABNF {{?RFC5234}}. The grammar is descriptive: it fixes the lexical shape a tool relies on, not every detail of the method names, option names, or payload syntax, which are drawn from the documents cited in {{relationship}}.

~~~~ abnf
message = start-line *option-line [blank-line payload]
start-line = (method SP request-target / code SP reason)
             [SP annot] newline
option-line = option-name ":" SP option-value newline
blank-line = newline
annot = "(" *annot-text ")"    ; free-form transport detail

method = 1*ALPHA                 ; e.g. GET, POST, iPATCH
request-target = path ["?" query] / absolute-URI
code = DIGIT "." 2DIGIT        ; e.g. 2.05
~~~~
{: #fig-grammar title="Descriptive grammar of the notation"}

An option line uses a single space after the colon. The blank line before a payload is present whenever a payload is present and omitted otherwise. A line that is not an option line and precedes a start line is a label. Transport annotations ({{transport}}) are free-form and are not required to be machine-parsed.

# Examples {#examples}

The examples in this section use a single sensor, `living-room-sensor`, throughout.

A GET on a resource that returns a plain-text representation:

~~~~ coap
GET /temperature

2.05 Content
Content-Format: 0 (text/plain)

22.3 Cel
~~~~
{: #fig-get title="GET with a text payload"}

A POST that creates a topic, carrying a CBOR payload in both directions:

~~~~ coap
POST /ps
Content-Format: TBD606 (application/core-pubsub+cbor)

{
  / topic-name /
  0: "living-room-sensor",
  / resource-type /
  2: "core.ps.data"
}

2.01 Created
Location-Path: /ps/h9392
Content-Format: TBD606 (application/core-pubsub+cbor)

{
  / topic-name /
  0: "living-room-sensor",
  / topic-data /
  1: "/ps/data/1bd0d6d",
  / resource-type /
  2: "core.ps.data"
}
~~~~
{: #fig-post title="POST creating a topic, with CBOR payloads"}

A FETCH that filters a collection, returning a CoRE Link Format response:

~~~~ coap
FETCH /ps
Content-Format: TBD606 (application/core-pubsub+cbor)

{
  / topic-name /
  0: "living-room-sensor",
  / resource-type /
  2: "core.ps.data"
}

2.05 Content
Content-Format: 40 (application/link-format)

</ps/2e3570>
~~~~
{: #fig-fetch title="FETCH with a CBOR filter"}

An iPATCH {{RFC8132}} that applies a partial update:

~~~~ coap
iPATCH /ps/h9392
Content-Format: TBD606 (application/core-pubsub+cbor)

{
  / observer-check /
  7: 7200
}

2.04 Changed
Content-Format: TBD606 (application/core-pubsub+cbor)

{
  / observer-check /
  7: 7200
}
~~~~
{: #fig-ipatch title="iPATCH applying a partial update"}

An Observe {{RFC7641}} subscription and its first notification:

~~~~ coap
GET /ps/data/1bd0d6d
Observe: 0

2.05 Content
Observe: 1234
Content-Format: 60 (application/cbor)

{
  / name /
  0: "living-room-sensor",
  / value /
  2: 22.5
}
~~~~
{: #fig-observe title="Observe subscription"}

Discovery of a broker through `.well-known/core` {{RFC6690}}:

~~~~ coap
GET /.well-known/core?rt=core.ps

2.05 Content
Content-Format: 40 (application/link-format)

<coaps://broker.example.com/ps>;rt="core.ps"
~~~~
{: #fig-discovery title="Discovery via .well-known/core"}

An ACE {{RFC9200}} token request and response:
<!-- carries a binary access token as a hexadecimal payload item: -->

~~~~ coap
POST /token
Content-Format: 19 (application/ace+cbor)

{
  / audience /
  5: "living-room-sensor",
  / scope /
  9: "read"
}

2.01 Created
Content-Format: 19 (application/ace+cbor)

{
  / access_token /
  1: h'8343a1010aa1054d',
  / token_type /
  34: 2,
  / profile /
  38: 2 / coap_oscore /
}
~~~~
{: #fig-ace title="ACE token request and response"}

An error response carries a diagnostic payload as a text string, without a Content-Format annotation, as described in {{Section 5.5.2 of RFC7252}}:

~~~~ coap
GET /ps/data/unknown

4.04 Not Found

Unknown topic
~~~~
{: #fig-error title="Error response with a diagnostic payload"}

# Relationship to Existing Notations {#relationship}

The notation composes elements that already appear in published documents rather than inventing new ones. The request line follows the method-and-URI form of {{RFC8132}} and {{RFC9176}}. The option lines follow the `Name: value` form used in {{RFC9203}}. The blank line separating options from the payload follows HTTP message syntax {{RFC9112}}. The Content-Format annotation, a number followed by a media type in parentheses, follows {{RFC8132}}. CBOR payloads follow the commented EDN style used in {{RFC9203}} and in CORECONF {{I-D.ietf-core-comi}}. Transport annotations use the terms of {{RFC7252}}.

{{survey}} presents the source notations in more detail and compares them along these dimensions.

# Tool Integration {#tool-integration}

The notation is structured so that authoring tools can parse and validate it. The mechanism below is one illustrative implementation, in kramdown-rfc; the notation does not depend on a single toolchain. In kramdown-rfc, an example is written in a `coap` sourcecode block:

~~~~
~~~ coap
POST /ps
Content-Format: TBD606 (application/core-pubsub+cbor)

{
  / topic-name /
  0: "living-room-sensor",
  / resource-type /
  2: "core.ps.data"
}
~~~
~~~~

A tool processes such a block in four steps: it parses the structure into the first line, the option lines, and the payload; it reads the content-format number from the Content-Format annotation; it dispatches the payload to a validator for that format, such as a CBOR diagnostic parser for CBOR-based formats or a JSON parser for JSON-based formats; and it reports any error with the line number within the block.

A block can reference a CDDL data definition so that the payload is checked for structure as well as well-formedness:

~~~~
~~~ coap
POST /ps
Content-Format: TBD606 (application/core-pubsub+cbor)

{
  / topic-name /
  0: "living-room-sensor",
  / resource-type /
  2: "core.ps.data"
}
~~~
{: check-cddl="pubsub-topic.cddl"}
~~~~

The building blocks for this validation already exist: kramdown-rfc validates JSON sourcecode blocks and has hooks for CBOR diagnostic types; the `edn-abnf` Ruby gem parses EDN (ABNF in {{I-D.ietf-cbor-edn-literals}}) and round-trips it to and from bytes; and the `cddl` Ruby gem validates CBOR against a CDDL data definition.

# Security Considerations {#seccons}

This notation appears in documentation, not on the wire, and it introduces no protocol behavior. It therefore adds no threat surface to the protocols whose exchanges it illustrates.

Examples written in this notation are illustrative and are not normative unless the surrounding document states otherwise. A reader who copies an example into an implementation is responsible for confirming that it matches the normative text.

Tooling that validates or executes example payloads processes input taken from a document under edit. Such tooling treats the payloads and any referenced CDDL data definition as untrusted input: a malformed EDN payload or a hostile data definition affects only the document build and does not reach a protocol endpoint.

# IANA Considerations {#iana}

This document has no IANA actions. The notation refers to CoAP option names, response codes, and content-format numbers by the values registered for them elsewhere; it defines no new code points, media types, or registries.

--- back

# Survey of Existing Notation Variants {#survey}

The notation in this document distills the conventions found in published RFCs and active drafts. This appendix records those source notations. Each is shown with a representative exchange and a one-line characterization.

Variant 1, from {{RFC7252}}, embeds message sequence charts with message-layer detail:

~~~~
Header: GET (T=CON, Code=0.01, MID=0x7d34)
Token: 0x20
Uri-Path: "temperature"

Header: 2.05 Content (T=ACK, Code=2.05, MID=0x7d34)
Token: 0x20
Payload: "22.3 C"
~~~~
{: #fig-v1 title="Variant 1 (RFC 7252): message sequence chart with transport detail"}

Variant 2, from {{RFC7641}}, shows the CoAP header as raw hexadecimal within numbered sequence diagrams:

~~~~
Header: GET 0x41011633
Token: 0x4a
Uri-Path: temperature
Observe: 0 (register)

Header: 2.05 0x61451633
Token: 0x4a
Observe: 9
Max-Age: 15
Payload: "18.5 Cel"
~~~~
{: #fig-v2 title="Variant 2 (RFC 7641): compact hex header"}

Variant 3, from {{RFC9176}}, uses `Req:`/`Res:` labels with the full URI on the first line:

~~~~
Req: POST coap://rd.example.com/rd?ep=node1
Content-Format: 40
Payload: </sensors/temp>;rt=temperature-c;if=sensor

Res: 2.01 Created
Location-Path: /rd/4521
~~~~
{: #fig-v3 title="Variant 3 (RFC 9176): Req/Res shorthand"}

Variant 4, from {{RFC8132}}, places the method and URI on the first line and lets the payload follow directly, with no explicit marker:

~~~~
FETCH CoAP://www.example.com/object
Content-Format: NNN (application/example-map-keys+json)
Accept: application/json

[ "foo" ]

2.05 Content
Content-Format: 50 (application/json)

{ "foo": ["bar","baz"] }
~~~~
{: #fig-v4 title="Variant 4 (RFC 8132): method + URI, payload follows"}

Variant 5, from {{RFC9203}}, prefixes each message with `Header:` and each body with `Payload:`, and carries CBOR in diagnostic notation:

~~~~
Header: POST (Code=0.02)
Uri-Host: "as.example.com"
Uri-Path: "token"
Content-Format: application/ace+cbor
Payload:

{
  / audience /
  5 : "tempSensor4711",
  / scope /
  9 : "read"
}
~~~~
{: #fig-v5 title="Variant 5 (RFC 9203): Header/Payload with CBOR diagnostic"}

Variant 6, from CORECONF {{I-D.ietf-core-comi}}, uses `REQ:`/`RES:` labels, an angle-bracketed path, and a parenthetical Content-Format:

~~~~
REQ: FETCH </c>
(Content-Format: application/yang-identifiers+cbor-seq)

1723, / current-datetime (SID 1723) /
[1533, "eth0"] / interface (SID 1533) with name = "eth0" /

RES: 2.05 Content
(Content-Format: application/yang-instances+cbor-seq)

{ 1723 : "2014-10-26T12:16:31Z" }
~~~~
{: #fig-v6 title="Variant 6 (CORECONF): REQ/RES with angle-bracket path"}

Variant 7, from the CoAP publish-subscribe work, follows the Variant 5 form with a path-only target and annotates the payload line with a format hint:

~~~~
Header: POST (Code=0.02)
Uri-Path: "ps"
Content-Format: TBD606 (application/core-pubsub+cbor)
Payload (in CBOR diagnostic notation):

{
  / topic-name /
  0: "living-room-sensor",
  / resource-type /
  2: "core.ps.data"
}

Header: Created (Code=2.01)
Location-Path: "ps"
Location-Path: "h9392"
Content-Format: TBD606 (application/core-pubsub+cbor)
Payload (in CBOR diagnostic notation):

{
  / topic-name /
  0: "living-room-sensor",
  / topic-data /
  1: "/ps/data/1bd0d6d",
  / resource-type /
  2: "core.ps.data"
}
~~~~
{: #fig-v7 title="Variant 7 (CoAP pub-sub): 9203-style with path-only URI"}

{{tab-variants}} compares the variants.

| Variant | First line | URI style | Payload marker | Payload format | Transport detail |
|---------|------------|-----------|----------------|----------------|------------------|
| 1 (7252) | `Header: GET (T=CON, Code=0.01, MID=...)` | Split options | `Payload:` inline | Plain text | Yes |
| 2 (7641) | `Header: GET 0x...` | Uri-Path option | `Payload:` inline | Plain text | Yes (hex) |
| 3 (9176) | `Req: POST coap://host/path` | Full URI | `Payload:` | Link-format | No |
| 4 (8132) | `FETCH CoAP://host/path` | Full URI | None (follows) | JSON | No |
| 5 (9203) | `Header: POST (Code=0.02)` | Split options | `Payload:` | CBOR diagnostic | No |
| 6 (COMI) | `REQ: FETCH </path>` | Path in `<>` | None (follows) | CBOR diagnostic | No |
| 7 (pubsub) | `Header: POST (Code=0.02)` | Split options | `Payload (...):` | CBOR diagnostic | No |
{: #tab-variants title="Comparison of existing CoAP notation variants"}

Three observations follow from the survey. Variants 5, 6, and 7 all carry CBOR diagnostic payloads but frame the surrounding message differently. Variants 3 and 4 are the closest to HTTP message syntax, while Variants 5 and 7 are the most structured and the most amenable to mechanical parsing. Only Variants 1 and 2 include message-layer information; the remaining variants describe the application layer alone.

[^todo1]

[^todo1]: TODO: Look at RFC 8323 and signaling codes (7.xx).

# Design Elements for a Unified Notation {#requirements}

The survey yields five design elements, which the notation in the body satisfies:

1. A first line that identifies the method for a request or the response code for a response, optionally with the URI.
2. Option lines as `Name: value` pairs.
3. An explicit separator between options and payload.
4. A declared payload format, as a Content-Format value or annotation, so that a tool can validate the payload.
5. A payload in the text notation appropriate to that format: CBOR EDN, JSON, link-format, or a hexadecimal dump.

# Acknowledgements
{: numbered='no'}

This notation distills conventions from {{RFC7252}}, {{RFC7641}}, {{RFC8132}}, {{RFC9176}}, {{RFC9203}}, and CORECONF, and from discussion in the CoRE Working Group.
