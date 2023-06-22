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

# Extension And Identifier

The RDAP extension identifier for this extension is "sc". This extension
defines the following JSON values may be found in an RDAP response:

* "sc_kind" 
* "sc_i18n"
* "sc_l10n"
* "sc_masked"
* "sc_geo"

Each is optionals, and each is described in a subseqent section.

# Kind

The "sc_kind" JSON value is a string, that is either "individual" or
"organization".

# Isomorphic Data

Some contact data has the same structure but different forms for different
purposes, depending on the context of data collection by the registry. This
data is in the "sc_i18n", "sc_l10n", and "sc_masked" values, which all
have the same internal structures.

"sc_l10n" is a localization of the data found in "sc_i18n". The method and type 
of localization is not defined in this document. In some instances, localization
is a process of transmutation of Unicode data to all ASCII data and in other
cases localization is a process of transliteration of data from one natural
language to another.

"sc_masked" is a masked version of the data found in "sc_i18n". This data is
used to describe methods of communication with the entity while partially or
fully masking identifying details of the entity.

## Names

Names of an entity may be given using either or all of the values named
"fullName", "organizationName", and "personName". The forms of each are
described below. 

Each is optional. When "fullName" is used without the others, it is
assumed to refer to an organization name or a person name. When "fullName"
is used with either of the others, it is assumed to refer to a person's
name. When "organizationName" is used with either or both "fullName" or
"personName", it is assumed the person is associated with the organization
being named.

    "fullName" : "Mr. Dr. Bob Lee Aloysius Smurd, Ph.d",
    "organizationName" : {
      "name" : "ACME",
      "subDivisions": [ "Floors and Windows", "Direct Sales" ],
      "suffixes" : [ "Gmbh" ]
    },
    "personName" : { 
      "prefixes": [ "Mr.", "Dr."],
      "firstNames": [ "Bob" ],
      "middleNames" : [ "Lee", "Aloysius" ],
      "lastNames" : [ "Smurd" ],
      "suffixes" : [ "Ph.d." ]
    } 


### Full Name

The full name of an entity expresses an unstructured name, and is a simple
JSON string with the name of "fullName":

    "fullName" : "Mr. Dr. Bob Lee Aloysius Smurd, Ph.d"

### Organization Name Components

An organization name can be expressed as a structured name with multiple components.
It is a JSON object with the name "organizationName" and has the following form:

    "organizationName" : {
      "name" : "ACME",
      "subDivisions": [ "Floors and Windows", "Direct Sales" ],
      "suffixes" : [ "Gmbh" ]
    }

The "name" string is required. Both "subDivisions" and "suffixes" are arrays
of strings, and both are optional. The "suffixes" array holds signifiers used
to convey legal or organization status.

If both "subDivisions" and "suffixes" are not used, the organization name
may be considered to be unstructured and the "name" string may hold components
of the organization name that would otherwise be expressed in the "subDivisions"
or "suffixes" arrays.

    "organizationName" : {
      "name" : "ACME Gmbh, Floors and Windows, Direct Sales"
    }

### Person Name Components

The expression of the components of person names uses a JSON object with the
name "personName" and has the following form:

    "personName" : { 
      "prefixes": [ "Mr.", "Dr."],
      "firstNames": [ "Bob" ],
      "middleNames" : [ "Lee", "Aloysius" ],
      "lastNames" : [ "Smurd" ],
      "suffixes" : [ "Ph.d." ]
    } 

Each component is an array of strings, and each is optional:

* "prefixes" holds honorifics, titles, and other signifiers that are
typically prefixed to a name.
* "firstNames" holds the set of names that preceed other names. These are often called "given" names.
* "middleNames" holds the set of names that succeed first names and preceed last names. These
are often referred to as "additional" names.
* "lastNames" holds the set of names that succeed other names. These are often called "sur" names.
* "suffixes" holds honorifics, titles, and other signifiers that are typically suffixed to a name.

## Postal Address

Postal addresses can be unstructured, structured, or semi-structured. A postal address
is expressed as a JSON object using the name "postalAddress". Components of the postal
address object are described below.

* address lines contain the street components and any other parts of a postal address
that cannot be more specifically expressed by other components. Address lines is
an array of strings, each string representing a line of the address, and is named
"lines".
* the "locality" value is a string representing the village, city, municipality, or similar
part of a postal address. This value is optional.
* the region name of a country, such as a state, province, or department, can be expressed as
a string named "regionName" and is optional.
* the region code, representing the ISO-3166-3 two letter code, can be expressed as
the JSON string named "regionCode" and is optional.
* the country name can be expressed as the JSON string named "countryName" and is optional.
* the ISO-3166-2 two letter code for a country can be expressed as the JSON string named
"countryCode".
* the postal code, sometimes referred to as a zip code or post code, may be expressed
by the optional JSON string named "postalCode".

The following is an example of a structured postal address:

    "postalAddress" : {
      "lines" : [ 
        "Suite 300",
        "123 Random Tree Name Street",
      ],
      "locality" : "Kalamazoo",
      "regionName" : "Michigan",
      "regionCode" : "MI",
      "countryName" : "United States of America",
      "countryCode" : "US",
      "postalCode" : "90215"
    }

The following is an example of an unstructured postal address:

    "postalAddress" : {
      "lines" : [ 
        "Suite 300",
        "123 Random Tree Name Street",
        "Kalamazoo, MI 90215"
      ],
    }

## Email Addresses

Email addresses can be expressed in a JSON array of strings named "emails". Each
string is a separate email address. If a string begins with "mailto:", the string
MUST be conformant to the mailto URI specified in [@!RFC6068]. Otherwise, the string
MUST be conformant to the address specification of [@!RFC5322, Section 3.4].
This JSON value is optional.

    "emails" : [
      "mailto:bob.smurd@example.com",
      "Bob Smurd <bobby@smurd.example.net>"
    ]

## Telephones

Telephones to be used for voice communication can be expressed in a JSON array of strings
named "voicePhones". Telephones to be used for facsimile machine communications can be
expressed in a JSON array of strings named "faxPhones". The string of each array signifies
a separate telephone number. Both JSON values are optional.

If a string begins with "tel:", the string MUST be conformant to the tel URI specified in
[@!RFC3966]. Otherwise the string is considered unstructured text. If possible, the
unstructurued text SHOULD be conformant to the [@!E.161] format and the [@!E.164] numbering
plan.

    "voicePhones" : [
      "tel:tel:+1-201-555-0123",
      "+447040202"
    ],
    "faxPhones" : [
      "tel:tel:+1-201-555-9999;ext=123"
    ]

## Web Contact

Communications with the entity using a web browser, often by submitting data via a web form,
can be expressed using a JSON array of strings called "webContacts". Each string MUST be
an https URI as specified by [@!RFC9110, Section 4.2.2]. This JSON value is optional.

    "webContacts" : [
      "https://example.com/contact-me"
    ]

## Organization Roles and Titles

To express the roles and titles of the entity within the organization, the "organizationalRoles"
and "organizationalTitles" JSON values may be used respectively. Each is an array of strings
containing unstructured text. Each is optional.

The "organizationalRoles" is not the same as the RDAP "roles" array. The RDAP "roles" array
describes a relationship between an entity it's containing object (e.g. domain, ip network).
An organizational role describes the relationship of an entity with the organization.

    "organizationalTtitles" : [
      "Senior Vice President",
      "Technical Fellow"
    ],
    "organizationalRoles" : [
      "Interim Chief Fun Officer",
      "Head of Squishy Toys"
    ]

## Language

The JSON value "lang" is the same as that defined by RDAP in [@!RFC9083, Section 4.4].
This JSON value is optional.

# Geographic Locations

Geographic locations can be expressed using the "sc_geo" JSON value, which is an array
of strings. Each string MUST be conformant to the geo URI scheme as defined in [@!RFC5870].

    "sc_geo" : [
      "geo:37.786971,-122.399677"
    ]

