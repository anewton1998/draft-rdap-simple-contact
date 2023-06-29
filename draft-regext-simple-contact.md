%%%
Title = "RDAP Simple Contact"
area = "Applications and Real-Time Area (ART)"
workgroup = "Registration Protocols Extensions (regext)"
abbrev = "rdap-simple-contact"
ipr= "trust200902"

[seriesInfo]
name = "Internet-Draft"
value = "draft-newton-regext-rdap-simple-contact-00"
stream = "IETF"
status = "info"
date = 2023-06-21T00:00:00Z

[[author]]
initials="A."
surname="Newton"
fullname="Andy Newton"
organization="ICANN"
[author.address]
email = "andy@hxr.us"

[[author]]
initials="T."
surname="Harrison"
fullname="Tom Harrison"
organization="APNIC"
[author.address]
email = "tomh@apnic.net"

%%%

.# Abstract

This document describes an extension to the Registry Data Access Protocol
for entity contact data using basic JSON values, objects, and arrays. The data
model defined by this document is purposefully limited to the data in-use by Internet
Number Registries and Domain Name Registries and does not attempt to model
the full data-set that can be expressed with other contact models such as jCard
or JSContact.

{mainmatter}

# Background

[@!RFC9083] defines the contact data of an entity using jCard ([@?RFC7095]),
which is a JSON format for vCard ([@?RFC6350]). Experience has shown that jCard
is unsuitable for RDAP because its "jagged" array style is unlike any other
JSON in RDAP; it is more difficutl to deserialize into objects that are easy
to work with, it is error prone and difficult to debug, and it can express
far more information than is necessary for RDAP.

This document defines the SimpleContact extension for RDAP. This extension
intends to model JSON is the same style and manner as other RDAP structures
and is purposefully limited to the data in-use by Internet Number Registries
and Domain Name Registries.

The purposeful limitation of the contact data model defined in this document
is informed by [@!RFC5733], the
[ICANN gTLD RDAP Response Profile](https://www.icann.org/en/system/files/files/rdap-response-profile-15feb19-en.pdf),
the [NRO RDAP Profile](https://bitbucket.org/nroecg/nro-rdap-profile/raw/v1/nro-rdap-profile.txt),
and [@!RFC7495].

# Simple Contact Extension And Identifier

The RDAP extension identifier for this extension is "sc". This extension
defines one JSON member named "sc_data" to be found in RDAP responses. 
"sc_data" is a JSON object, and it has child members described in the following
sections. Each child member of "sc_data" is optional.

There are two common, optional JSON members of these child members: "lang" and "masked".

The JSON member "lang" is the same as that defined by RDAP in [@!RFC9083, Section 4.4].

The JSON member "masked" is a boolean. When true, this indicates that the data of the
JSON object provided is to facilitate communication with the entity in a manner that
hides or obfuscates the identity of the entity. This serves the same purpose as
the vCard properties defined in [@!RFC8605].

Most of the child members are arrays allowing the expression of multiple
variants of the same information. The order in which items appear in these
arrays denotes preference order for the variants.

## Kind

The "kind" JSON value is a string, that is either "individual", "role" or
"organization". An example:

    "kind" : "role"


## Names

Names may be expressed as unstructured text and as structured data.
When names are given as structured data, it is RECOMMENDED to also
give unstructured names. This may require servers which store names
in structured data to synthesize the unstructured names. 
Servers which store only unstructured names SHOULD NOT attempt to synthesize 
and provide structured names from those values, because of the difficulty 
of doing this correctly for all types of names."

Names can be expressed for each kind of the entity, as described in the
"kind" string. When describing an "individual", the name of the individual's
role and organization may also be expressed. When describing a "role", the
name of the role's organization may also be expressed. It is NOT RECOMMENDED
to express the name of a role or individual when the kind is "organization".

Each type of name is expressed as a JSON array in which each item is an
object with the following optional members:

* "name" - the unstructured name as a string
* "lang" - see above
* "masked" - see above
* "parts" - a JSON object that varies for each type of name.

A> The co-authors of this specification are skeptical of the need for
A> structured names in RDAP. None of the references in Section 1 indicate
A> that names are collected or published structurally. Additionally,
A> attempts to structure names for all contexts and languages is unlikely
A> to yield a complete solution 
A> (see https://mailarchive.ietf.org/arch/msg/calsify/3MR_gLO-1NuyQvE8DIl88KNglSs/).

### Individual Names

The name of an individual may be expressed with the JSON member named
"individualNames". The "parts" object for "individualNames" has the
following optional members:

* "prefixes" - an array of strings holding honorifics, titles, and other signifiers that are
typically prefixed to a name.
* "firstNames" - an array of strings holding the set of names that preceed other names. These are often called "given" names.
* "middleNames" - an array of strings holding the set of names that succeed first names and preceed last names. These
are often referred to as "additional" names.
* "lastNames" - an array of strings holding the set of names that succeed other names. These are often called "sur" names.
* "suffixes" - an array of strings holding honorifics, titles, and other signifiers that are typically suffixed to a name.

The following is an example:

    "individualNames" : [
      {
        "name" : "Dr. Bob Lee Aloysius Smurd, Ph.d.",
        "lang" : "en-AU",
        "parts" : {
          "prefixes": [ "Dr."],
          "firstNames": [ "Bob" ],
          "middleNames" : [ "Lee", "Aloysius" ],
          "lastNames" : [ "Smurd" ],
          "suffixes" : [ "Ph.d." ]
        }
      }
    ]

### Role Names

The name of a role may be expressed with the JSON member named
"roleNames". The "parts" object for "roleNames" has one 
member named "roles" which is an array of strings.

The following is an example:

    "roleNames" : [
      {
        "name" : "Abuse Prevention, Trust and Safety",
        "lang" : "en-AU",
        "parts" : {
          "roles" : [ "Abuse Prevention", "Trust and Safety"],
        }
      }
    ]

### Organization Names

The name of an organization may be expressed with the JSON member named
"organizationNames". The "parts" object for "organizationNames" has the
following optional members:

* "name" - a string holding the name of the organization without other qualifiers.
* "subDivisions" - an array of strings, each the name of a subdivision of the organization.
* "suffixes" - an array of strings, each signifying a legal status of the organization.
* "legalId" - an array of strings, each containing a legal identifier of the organization.

The following is an example:

    "organizationNames" : [
      {
        "name" : "ACME Pty, Floors and Windows, Direct Sales",
        "lang" : "en-AU",
        "parts" : {
          "name" : "ACME",
          "subDivisions": [ "Floors and Windows", "Direct Sales" ],
          "suffixes" : [ "Pty" ],
        }
      }
    ]

## Postal Addresses

Like names, postal addresses can be expressed as either structured or unstructured.
When postal addresses are given as structured data, it is RECOMMENDED to also
give unstructured addresses. This may require servers which store addresses
in structured data to synthesize the unstructured addresses. 
Servers which store only unstructured addresses SHOULD NOT attempt to synthesize 
and provide structured addresses from those values, because of the difficulty of 
doing this correctly for all types of addresses.

Postal addresses are expressed with the "postalAddresses" JSON member, which is an
array of objects each with the following optional members:

* "completeAddress" - holds the unstructured postal address as an array of strings
in which each string represents a line of a postal address.
* "deliveryLines" - an array of strings in which each string represents a line of a
postal address specific to delivery within the locality of the address. This information
typically contains street names and numbers, apartment or suite names, and other information
necessary for the delivery of postal mail within a locality.
* "locality" - a string representing the village, city, municipality, or similar
part of a postal address.
* "regionName" - a string of the region name of a country, such as a state, province, or department.
* "regionCode" - a string representing the ISO-3166-3 two letter code.
* "countryName" - a string containing the country name.
* "countryCode" - a string containing the ISO-3166-2 two letter code for the country.
* "postalCode" - a string containing the postal code, sometimes referred to as a zip code or post code.
* "lang" - see above
* "masked" - see above

The following is an example of a postal address:

    "postalAddresses" : [
      {
        "completeAddress" : [
          "Suite 300",
          "123 Random Tree Name Street",
          "Kalamazoo, MI 90125 US"
        ]
        "deliveryLines" : [ 
          "Suite 300",
          "123 Random Tree Name Street",
        ],
        "locality" : "Kalamazoo",
        "regionName" : "Michigan",
        "regionCode" : "MI",
        "countryName" : "United States of America",
        "countryCode" : "US",
        "postalCode" : "90215",
        "lang" : "en-US"
      }
    ]


## Email Addresses

Email addresses can be expressed in a JSON array of objects named "emails". Each
object contains the following members:

* "email" - a string containing the email address.
* "lang" - optional, see above
* "masked" - optional, see above

If the string in "email" begins with "mailto:", the string
MUST be conformant to the mailto URI specified in [@!RFC6068]. Otherwise, the string
MUST be conformant to the address specification of [@!RFC5322, Section 3.4].
This JSON value is optional.

    "emails" : [
      {
        "email": "山田太郎 <yamada.taro@example.net>",
        "lang" : ja
      },
      {
        "email" : "Yamada Taro <yamada.taro@example.net>"
        "lang" : "ja-Latn"
      }
    ]

## Telephones

Telephones to be used for voice communication can be expressed in a JSON array of objects
named "voicePhones". Telephones to be used for facsimile machine communications can be
expressed in a JSON array of objects named "faxPhones". Each object has the following 
members:

* "phone" - a string holding the phone number
* "masked" - optional, see above

If the string in "phone" begins with "tel:", the string MUST be conformant to the tel URI specified in
[@!RFC3966]. Otherwise the string is considered unstructured text. If possible, the
unstructurued text SHOULD be conformant to the [@!E.161] format and the [@!E.164] numbering
plan.

The following are examples:

    "voicePhones" : [
      {
        "phone": "tel:+1-201-555-0123",
        "masked" : false
      },
      {
        "phone" : "+447040202",
      }
    ],
    "faxPhones" : [
      {
        "phone" : "tel:+1-201-555-9999;ext=123",
        "masked" : false
      }
    ]

## Web Contacts

Communications with the entity using a web browser, often by submitting data via a web form,
can be expressed using a JSON array of objects called "webContacts". Each object has the following
members:

* "uri" - a string with an HTTPS URI as specified by [@!RFC9110, Section 4.2.2].
* "masked" - optional, see above.

An example:

    "webContacts" : [
      {
        "uri" : "https://example.com/contact-me"
      }
    ]

## Geographic Locations

Geographic locations can be expressed using the "geo" JSON value, which is an array
of strings. Each string MUST be conformant to the geo URI scheme as defined in [@!RFC5870].

    "geo" : [
      "geo:37.786971,-122.399677"
    ]

"geo" values SHOULD NOT be used with contact data when "kind" is "individual".

# EPP Int/Loc Data

[@!RFC5733] defines mechanisms to indicate data is "localized" or "internationalized"
using "int" and "loc" types. The "int" designates data as being 7-bit ASCII. To express
contact data with the "int" designation in SimpleContact, it is RECOMMENDED that a language
tag with the "Latn" script subtag (see [!@RFC5646]) be used.

# No jCard Extension and Identifier

This document also defines a second RDAP extension to signal the non-use of jCard in RDAP
responses. The identifier for this extension is "noJCard". When used with the RDAP-X
media type [@!I-D.draft-newton-regext-rdap-x-media-type] and the SimpleContact identifier, 
a client can signal to a server that entities should substitute SimpleContact data for jCard data.
When used with queries containing large amounts of entity data, such as RDAP searches, this
can reduce the payload significantly (instead of sending both jCard and SimpleContact).

Use of the "noJCard" extension is independent of the SimpleContact extension, and may be
used for other purposes as is appropriate to server policy.

{backmatter}
