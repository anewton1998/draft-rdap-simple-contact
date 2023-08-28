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
status = "standard"
date = 2023-08-25T00:00:00Z

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
intends to model JSON in the same style and manner as other RDAP structures
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
the vCard properties defined in [@!RFC8605]. This JSON member is not intended to
redact contact information but rather provide a means of specifying contact
information that is useable (e.g. a working email address) that does not yield
the identity of the contact. Marking contact data as "masked" signifies to the
client that communications using that data may be through an intermediary or
other indirect means.

Most of the child members are arrays allowing the expression of multiple
variants of the same information. The order in which items appear in these
arrays denotes preference order for the variants.

## Kind

The "kind" JSON value is a string, that is either "individual", "role" or
"organization". An example:

    "kind" : "role"

There is no equivalent to the "role" value in either jCard or JSContact,
though role entities are common in RDAP registries.

## Names

Names can be expressed for each kind of the entity, as described in the
"kind" string. When describing an "individual", the name of the individual's
role and organization may also be expressed. When describing a "role", the
name of the role's organization may also be expressed. It is NOT RECOMMENDED
to express the name of a role or individual when the kind is "organization".

Names are expressed using the "individualNames",
"roleNames", and "organizationNames" JSON members for individuals, roles,
and organizations respectively. The value of each is an array in which
each item is an object with the following members:

* "name" - unstructured textual name as a string
* "lang" - optional, see above
* "masked" - optional, see above

The following is an example:

    "individualNames" : [
      {
        "name" : "Dr. Bob Lee Aloysius Smurd, Ph.d.",
        "lang" : "en-AU"
      }
    ],
    "roleNames" : [
      {
        "name" : "Abuse Prevention, Trust and Safety",
        "lang" : "en-AU"
      }
    ],
    "organizationNames" : [
      {
        "name" : "ACME Pty",
        "lang" : "en-AU"
      }
    ]

RDAP allows the expression of nested entities as the entity object class has its
own `entities` array. Some servers express the relationship of individuals to roles
and/or organizations by nesting entities inside other entities. SimpleContact does
not remove this capability nor prohibit it. However, nesting of entities is NOT
RECOMMENDED if the expression of a relationship between an individual and a role
or an organization can be accomplished using names alone due to the complexity
in representation of those relationships by a client. If a server is to express
an individual with a relationship to a role and/or organization and each have
differences other than names (e.g. separate postal addresses), then nesting is
RECOMMENDED.

## Postal Addresses

Postal addresses can be expressed as a series of strings, each representing a
separate line of text as it would appear on an item for delievery in a postal
system. Additionally, postal addresses may be augmented with some of the common
fields found in postal systems for the purposes of processing these addresses
in non-postal systems.

Postal addresses are expressed with the "postalAddresses" JSON member, which is an
array of objects each with the following optional members:

* "address" - holds the unstructured postal address as an array of strings
in which each string represents a line of a postal address.
* "cc" - a string containing the ISO-3166-2 code.
* "pc" - a string containing the postal code, sometimes referred to as a zip code or post code.
* "lang" - see above
* "masked" - see above

The following is an example of a postal address:

    "postalAddresses" : [
      {
        "address" : [
          "Suite 300",
          "123 Random Tree Name Street",
          "Kalamazoo, MI 90125 US"
        ]
        "cc" : "US-MI",
        "pc" : "90215",
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
MUST be conformant to the address specification of [@!RFC6531, Section 3.3].
This JSON value is optional.

    "emails" : [
      {
        "email": "山田太郎@example.net",
        "lang" : "ja"
      },
      {
        "email" : "yamada.taro@example.net",
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
unstructurued text SHOULD be conformant to the [@!ITU.E161.2001] format and the [@!ITU.E164.1991] numbering
plan.

The following are examples:

    "voicePhones" : [
      {
        "phone": "tel:+1-201-555-0123",
        "masked" : false
      },
      {
        "phone" : "+447040202"
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

## An Entity Example {#example}

The following is an example of an RDAP entity using SimpleContact:

    {
      "objectClassName" : "entity",
      "handle":"XXXX",
      "sc_data": {
        "kind" : "individual",
        "individualNames" : [
          {
            "name" : "山田太郎",
            "lang" : "ja"
          },
          {
            "name" : "Yomada Taro",
            "lang" : "en"
          }
        ],
        "roleNames" : [
          {
            "name" : "登録サービス ヘルプデスク",
            "lang" : "ja"
          },
          {
            "name" : "Registration Services Help Desk",
            "lang" : "en"
          }
        ],
        "organizationNames" : [
          {
            "name" : "アクメ",
            "lang" : "ja"
          },
          {
            "name" : "ACME",
            "lang" : "en"
          }
        ],
        "postalAddresses" : [
          {
            "address" : [
              "〒150-2345 東京都渋谷区本町2丁目4-7サニーマンション203",
            ]
            "cc" : "JP-13",
            "pc" : "150-2345",
            "lang" : "ja"
          },
          {
            "address" : [
              "Sunny Mansion #203",
              "4-7 Hommachi 2-choume",
              "Shibuya-ku, TOKYO 150-2345"
            ]
            "cc" : "JP-13",
            "pc" : "150-2345",
            "lang" : "en"
          }
        ],
        "emails" : [
          {
            "email": "山田太郎@example.net",
            "lang" : "ja"
          },
          {
            "email" : "yamada.taro@example.net",
            "lang" : "ja-Latn"
          }
        ],
        "voicePhones" : [
          {
            "phone" : "+81(03) 1234-5678",
            "masked" : true
          }
        ],
        "faxPhones" : [
          {
            "phone" : "tel:+810312345679"
          }
        ],
        "webContacts" : [
          {
            "uri" : "https://example.net/contact-me"
          }
        ]
      },
      "roles":[ "registrar" ],
      "publicIds":[
        {
          "type":"IANA Registrar ID",
          "identifier":"1"
        }
      ],
      "remarks":[
        {
          "description":[
            "She sells sea shells down by the sea shore.",
            "Originally written by Terry Sullivan."
          ]
        }
      ],
      "links":[
        {
          "value":"https://example.com/entity/XXXX",
          "rel":"self",
          "href":"https://example.com/entity/XXXX",
          "type" : "application/rdap+json"
        },
        {
          "value":"https://example.com/entity/XXXX",
          "rel":"about",
          "href":"https://example.com/entity/XXXX.vcard",
          "type" : "text/vcard"
        }
      ],
      "events":[
        {
          "eventAction":"registration",
          "eventDate":"1990-12-31T23:59:59Z"
        }
      ],
      "asEventActor":[
        {
          "eventAction":"last changed",
          "eventDate":"1991-12-31T23:59:59Z"
        }
      ]
    }

# EPP Int/Loc Data

[@!RFC5733] defines mechanisms to indicate data is "localized" or "internationalized"
using "int" and "loc" types. The "int" designates data as being 7-bit ASCII. To express
contact data with the "int" designation in SimpleContact, it is RECOMMENDED that a language
tag with the "Latn" script subtag (see [@!RFC5646]) be used.

# Links to Other Contact Formats

SimpleContact does not attempt to model all possible forms of contact formats or data.
Where an RDAP server can provide a more extensive form such as vCard, jCard, or JSContact,
these can be expressed in an RDAP link using the "about" rel type and the media type
appropriate to that form. section (#example) contains an example using vCard.

RDAP servers MUST place the same access restrictions upon these resources as they do
on the RDAP entity from which they are referenced. It is NOT RECOMMENDED to link to
contact data provided by other servers or servers under separate authorities.

# The No jCard Extension and Identifier

This document also defines a second RDAP extension to signal the non-use of jCard in RDAP
responses. The identifier for this extension is "noJCard". When used with the RDAP-X
media type [@!I-D.newton-regext-rdap-x-media-type] and the SimpleContact identifier, 
a client can signal to a server that entities should substitute SimpleContact data for jCard data.
When used with queries containing large amounts of entity data, such as RDAP searches, this
can reduce the payload significantly (instead of sending both jCard and SimpleContact).

Use of the "noJCard" extension is independent of the SimpleContact extension, and may be
used for other purposes as is appropriate to server policy.

{backmatter}

<reference anchor="ITU.E164.1991">
<front>
<title>
The International Public Telecommunication Numbering Plan
</title>
<author>
<organization>International Telecommunications Union</organization>
</author>
<date year="1991"/>
</front>
<seriesInfo name="ITU-T" value="Recommendation E.164"/>
</reference>

<reference anchor="ITU.E161.2001">
<front>
<title>
Arrangement of digits, letters and symbols on telephones and other devices that can be used for gaining access to a telephone network
</title>
<author>
<organization>International Telecommunications Union</organization>
</author>
<date year="2001"/>
</front>
<seriesInfo name="ITU-T" value="Recommendation E.161"/>
</reference>

