# Environment Query Language (EQL)
<br />

## Introduction
Environment Query Language (EQL) defines a common query language and ontology for military simulation environments. Implementations are responsible for mapping their internal data structures to the standard EQL filter groups and values.

EQL is designed to abstract away the developers notion or Intel, Units, Groups etc. so consumers of various systems can write the same "find command" and it works everywhere.  
<br />

## Usage
EQL is designed to mimic plain English from a traditional military setting.  It follows a very simple structure:

   `ACTION | TARGET | COMPLEX FILTERS <| [ALSO] | [COMPLEX FILTERS]>`

Complex filters take the form of a Filter Group keyword, followed by one or more known Filter Value keywords.  They can contain multiple Filter Group Keywords per request:

   `DIMENSION Land Air IDENTITY Hostile`

<br />


## Technical Implementation
### Required Data Types
* String
* Integer
* Double

*Note: Boolean is implicitly implied by the presence or exclusion of a filter value*

<br />

### Implementation
Commands and values are separated by a single white-space.  All Commands and available values must be UTF-8 encoded strings, containing no white-space.  All string values are case invariant.  The following symbols are also illegal in command or value inputs:
* Asterisk (*)
* Pipe (|)
* Square Open ([)
* Square Close (])

<br />

### Syntax
#### Action
Every EQL command starts with an "Action" - one of the ACTION Filter Groups:

	    `FIND | HIDE | COUNT | COUNTNOT`

This defines what the query will do with the full dataset:
* FIND - Finds all TARGETS that match any Complex Filters in the query
* HIDE - Finds all TARGETS that DO NOT match any Complex Filters in the query
* COUNT - Returns the total number of TARGETS that match any Complex Filters in the query
* COUNTNOT- Returns the total number TARGETS that DO NOT match any Complex Filters in the query

<br />

#### Target
This will always be followed by the "Target" - one of the TARGET Filter Groups:

	    `UNIT | GROUP | LOCATION `

Defines what you are expecting back from the system being queried.
* UNIT - Request singular entities matching filters within the environment
* GROUP - Request the "parent structures" of entities matching filters within the environment
* LOCATION - Request coordinate values of matching filters in the environment

A value after the used Target keyword will always be a Filter Group

<br />

#### Complex Filters
Multiple Filter Groups can be used together and multiple Filter Values can be used for each Filter Group.  A Filter Group is processed as a whole.  Multiple Complex Filters can be separated by "ALSO".  A combination of multiple Filter Groups, with Filter Values makes a Complex Filter. 

Filter Values are always assessed with an 'or' condition:

	`DIMENSION Land [or] Air [or] Sea`

Filter Groups are always assessed with an "and" condition:

	`DIMENSION Land [or] Air [and] IDENTITY Hostile`

<br />

##### Filter Combination
Multiple Complex Filters can be chained using the `ALSO` keyword.  A Query matches if any Complex Filter matches.  For example:

`[FILTER_GROUP] [FILTER_VALUE] .. [FILTER_GROUP] [FILTER_VALUE] .. [ALSO] [FILTER_GROUP] [FILTER_VALUE] .. [FILTER_GROUP] [FILTER_VALUE] ..`

*Note that following the ALSO keyword, another Complex Filter must immediately follow, NOT an Action keyword.*

<br />

##### Filter Parsing
Filter Groups are self-delimiting. A Filter Group ends when another Filter Group keyword, the ALSO keyword, or the end of the query is encountered.

<br />

##### Query Evaluation
1) A Filter Group matches when any Filter Value within that group matches
2) A Complex Filter matches when all Filter Groups within that Complex Filter match
3) A Query matches when any Complex Filter within the Query matches

<br />

##### Filter Groups and Values

###### Filter Group: IDENTITY

Valid Filter Values:
* Pending
* Unknown
* Assumed_Friend
* Neutral
* Friend
* Suspect
* Hostile
* Info

<br />

###### Filter Group: DIMENSION

Valid Filter Values:
* Air
* Land
* Sea
* Space
* Subsurface

<br />

###### Filter Group: STATE

Valid Filter Values:
* Active
* Inactive
* Hidden
* Static
* Moving

<br />

###### Filter Group: COUNTRY

Filter values MUST be a STANAG 1059 Edition 9 compliant 3 letter code.  See https://github.com/odskee/MilDocs/blob/main/STANAG%201059.pdf for more details.

<br />


###### Filter Group: CLASSIFICATION

Filter values MUST be a 6 digit integer representing Entity, Entity Type and Entity Subtype as defined in MIL-STD-2525D Appendix A.5.4 Set B.  Values MUST NOT be longer than 6 digits and will not include Modifier 1 or Modifier 2 as detailed in MIL-STD-2525D.  See https://a44074a4-8852-4e80-9d62-09098560709e.s3.eu-north-1.amazonaws.com/ArchitectsLounge/MIL-STD-2525D.pdf for more information.

<br />

###### Filter Group: LOCATION

This is a 'parameterised' filter:

    LOCATION [WITHIN | NOTWITHIN | ABOVE | BELOW] Distance [OF Latitude/Longitude]

Following the LOCATION keyword, one the following Valid Filter Values MUST come next:
* WITHIN
* NOTWITHIN
* ABOVE
* BELOW

This must then be followed by, in order and separated with White-Space:
1) A decimal number represented as Meters.  If the keyword preceding this is WITHIN or NOTWITHIN, then the next steps are required.  If the keyword preceding this is ABOVE or BELOW, the decimal number terminates the Location Value. 
2) The keyword "OF".
3) Two decimal point numbers representing Decimal Degrees in the format [Latitude Longitude].

<br />

###### Filter Group: CAPABILITY

This is a simplification of "Environment Role":
* Attack
* Defend
* Support

<br />


## Compliance
An implementation is considered EQL compliant if it:
1.  Supports all required Actions.
2.  Supports all required Targets.
3.  Correctly evaluates Filter Groups according to the Query Evaluation rules.
4.  Correctly maps internal data structures to EQL Filter Groups and Filter Values.
5.  Treats commands and values as case-insensitive.
6.  An implementation SHOULD declare the highest EQL version it supports.
<br />

## Versioning

EQL follows Semantic Versioning (SemVer).  Version numbers take the form:

MAJOR.MINOR.PATCH

Examples:

1.0.0
1.1.0
2.0.0

<br />

### Major Version

A major version change indicates a breaking change to the EQL specification.  Implementations compliant with one major version are not guaranteed to be compatible with another major version.  Examples include:

* Removal of a Filter Group
* Removal of a Filter Value
* Changes to query evaluation behaviour
* Changes to grammar or syntax

<br />

### Minor Version

A minor version change indicates backwards-compatible additions.  Examples include:

* New Filter Groups
* New Filter Values
* Additional Actions
* Clarifications to existing behaviour

<br />

### Patch Version

A patch version change indicates documentation updates, clarifications, corrections, or non-functional changes.  Patch releases MUST NOT modify the behaviour of valid EQL queries.

<br />

### Example

EQL Version: 1.0.0

<br />

## Error Handling
Implementations MUST reject invalid EQL queries and MUST NOT attempt to infer, correct, or modify invalid queries.

### Invalid Queries
A query is invalid if:

* An unknown Action is specified
* An unknown Target is specified
* An unknown Filter Group is specified
* An unknown Filter Value is specified
* A parameterised filter is malformed
* Required parameters are missing
* Keywords appear in an invalid order
* A query terminates unexpectedly

<br />

### Error Reporting
Implementations SHOULD provide a human-readable error message indicating:
* The position of the error
* The offending token
* The reason the query is invalid

<br />

## Grammar
### Query

*ACTION TARGET [COMPLEX_FILTER]*  

<br />

### Query

*ACTION TARGET [COMPLEX_FILTER] ALSO [COMPLEX_FILTER]*  

<br />

### Complex Filter

*FILTER_GROUP FILTER_VALUE FILTER_GROUP FILTER_VALUE*

<br />

### Complex Filter

*FILTER_GROUP FILTER_VALUE FILTER_VALUE FILTER_GROUP FILTER_VALUE*

<br />

### Filter Groups

IDENTITY<br />
DIMENSION<br />
STATE<br />
LOCATION<br />
CAPABILITY<br />

<br />

### Filter Values

PENDING<br />
UNKNOWN<br />
ASSUMED_FRIEND<br />
NEUTRAL<br />
FRIEND<br />
SUSPECT<br />
HOSTILE<br />
INFO<br />
AIR<br />
LAND<br />
SEA<br />
SPACE<br />
SUBSURFACE<br />
ACTIVE<br />
INACTIVE<br />
HIDDEN<br />
STATIC<br />
MOVING<br />
ATTACK<br />
DEFEND<br />
SUPPORT<br />
<br />



## Example

Find any friendly land and air groups within 10KM of Damascus that are active and can attack along with any hostile land groups within 5KM of Damascus:

`FIND GROUP STATE Active IDENTITY Friendly LOCATION WITHIN 10000 OF 33.513 36.276 DIMENSION Land Air CAPABILITY ATTACK ALSO IDENTITY Hostile LOCATION WITHIN 5000 OF 33.513 36.276`

