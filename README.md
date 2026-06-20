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

###### Filter Group: LOCATION

This is a 'parameterised' filter:

    LOCATION [INSIDE | OUTSIDE | WITHIN] Distance OF Latitude/Longitude

Following the LOCATION keyword, one the following Valid Filter Values MUST come next:
* INSIDE
* OUTSIDE
* WITHIN

This must then be followed by, in order and separated with White-Space:
1) A whole number represented as Meters
2) The keyword "OF"
3) Two decimal point numbers representing Decimal Degrees in the format [Latitude Longitude] 

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

