# Chapter 13 - Mining Text to Find Meaningful Data

## Introduction

Text mining is a powerful technique to extract structured data from unstructured text. This chapter explores how SQL can be used to analyze text in speeches, reports, and other documents, transforming unstructured information into valuable insights. We'll explore string functions, pattern matching with regular expressions, and PostgreSQL's advanced full-text search capabilities.

## Formatting Text Using String Functions

PostgreSQL provides over 50 built-in string functions for text manipulation. These functions handle routines such as capitalization, string combination, and space removal.

### Case Formatting

```sql
-- PostgreSQL Standard (snake_case)
SELECT upper('hello');                   -- Returns 'HELLO'
SELECT lower('Randy');                   -- Returns 'randy'
SELECT initcap('at the end of the day'); -- Returns 'At The End Of The Day'

-- Domain Driven Design (PascalCase)
SELECT upper('hello') FROM "Text"."Sample";                   -- Returns 'HELLO'
SELECT lower('Randy') FROM "Text"."Sample";                   -- Returns 'randy' 
SELECT initcap('at the end of the day') FROM "Text"."Sample"; -- Returns 'At The End Of The Day'
```

These functions transform text case:
- `upper()` capitalizes all alphabetical characters
- `lower()` converts text to lowercase
- `initcap()` capitalizes the first letter of each word (PostgreSQL-specific)

### Character Information

```sql
-- PostgreSQL Standard (snake_case)
SELECT char_length(' Pat ');              -- Returns 5
SELECT position(', ' IN 'Tan, Bella');    -- Returns 4

-- Domain Driven Design (PascalCase)
SELECT char_length(' Pat ') FROM "Text"."Sample";              -- Returns 5
SELECT position(', ' IN 'Tan, Bella') FROM "Text"."Sample";    -- Returns 4
```

These functions return data about the string:
- `char_length()` counts characters including spaces
- `position()` finds the location of a substring within a string

### Removing Characters

```sql
-- PostgreSQL Standard (snake_case)
SELECT trim('s' FROM 'socks');            -- Returns 'ock'
SELECT trim(trailing 's' FROM 'socks');   -- Returns 'sock'
SELECT trim(' Pat ');                     -- Returns 'Pat'
SELECT char_length(trim(' Pat '));        -- Returns 3
SELECT ltrim('socks', 's');               -- Returns 'ocks'
SELECT rtrim('socks', 's');               -- Returns 'sock'

-- Domain Driven Design (PascalCase)
SELECT trim('s' FROM 'socks') FROM "Text"."Sample";            -- Returns 'ock'
SELECT trim(trailing 's' FROM 'socks') FROM "Text"."Sample";   -- Returns 'sock'
SELECT trim(' Pat ') FROM "Text"."Sample";                     -- Returns 'Pat'
SELECT char_length(trim(' Pat ')) FROM "Text"."Sample";        -- Returns 3
SELECT ltrim('socks', 's') FROM "Text"."Sample";               -- Returns 'ocks'
SELECT rtrim('socks', 's') FROM "Text"."Sample";               -- Returns 'sock'
```

The `trim()` function removes specified characters:
- Without parameters, it removes spaces
- With parameters, it removes specified characters
- Can remove leading, trailing, or both with keywords
- `ltrim()` and `rtrim()` are PostgreSQL-specific variants for left/right trimming

### Extracting and Replacing Characters

```sql
-- PostgreSQL Standard (snake_case)
SELECT left('703-555-1212', 3);        -- Returns '703'
SELECT right('703-555-1212', 8);       -- Returns '555-1212'
SELECT replace('bat', 'b', 'c');       -- Returns 'cat'

-- Domain Driven Design (PascalCase)
SELECT left('703-555-1212', 3) FROM "Text"."Sample";        -- Returns '703'
SELECT right('703-555-1212', 8) FROM "Text"."Sample";       -- Returns '555-1212'
SELECT replace('bat', 'b', 'c') FROM "Text"."Sample";       -- Returns 'cat'
```

These functions extract or substitute characters:
- `left()` extracts characters from the left side
- `right()` extracts characters from the right side
- `replace()` substitutes characters in a string

## Matching Text Patterns with Regular Expressions

Regular expressions provide a powerful notational language to describe and match text patterns.

### Regular Expression Notation

| Expression | Description |
|------------|-------------|
| `.` | A dot is a wildcard that finds any character except a newline |
| `[FGz]` | Any character in the square brackets. Here, F, G, or z |
| `[a-z]` | A range of characters. Here, lowercase a to z |
| `[^a-z]` | The caret negates the match. Here, not lowercase a to z |
| `\w` | Any word character or underscore. Same as `[A-Za-z0-9_]` |
| `\d` | Any digit |
| `\s` | A space |
| `\t` | Tab character |
| `\n` | Newline character |
| `\r` | Carriage return character |
| `^` | Match at the start of a string |
| `$` | Match at the end of a string |
| `?` | Get the preceding match zero or one time |
| `*` | Get the preceding match zero or more times |
| `+` | Get the preceding match one or more times |
| `{m}` | Get the preceding match exactly m times |
| `{m,n}` | Get the preceding match between m and n times |
| `a|b` | The pipe denotes alternation. Find either a or b |
| `( )` | Create and report a capture group or set precedence |
| `(?:)` | Negate the reporting of a capture group |

Regular expression examples on the text "The game starts at 7 p.m. on May 2, 2019.":

| Expression | What it matches | Result |
|------------|-----------------|--------|
| `.+` | Any character one or more times | The game starts at 7 p.m. on May 2, 2019. |
| `\d{1,2} (?:a.m.\|p.m.)` | One or two digits followed by a space and either a.m. or p.m. | 7 p.m. |
| `^\w+` | One or more word characters at the start | The |
| `\w+.$` | One or more word characters followed by any character at the end | 2019. |
| `May\|June` | Either of the words May or June | May |
| `\d{4}` | Four digits | 2019 |
| `May \d, \d{4}` | May followed by a space, digit, comma, space, and four digits | May 2, 2019 |

You can use pattern matching in PostgreSQL with the `substring()` function:

```sql
-- PostgreSQL Standard (snake_case)
SELECT substring('The game starts at 7 p.m. on May 2, 2019.' from '\d{4}');
-- Returns: 2019

-- Domain Driven Design (PascalCase)
SELECT substring('The game starts at 7 p.m. on May 2, 2019.' from '\d{4}') 
FROM "Text"."Sample";
-- Returns: 2019
```

## Turning Text to Data with Regular Expression Functions

### Creating a Table for Crime Reports

Let's create a table to store crime reports as an example of text processing:

```sql
-- PostgreSQL Standard (snake_case)
CREATE TABLE crime_reports (
    crime_id bigserial PRIMARY KEY,
    date_1 timestamp with time zone,
    date_2 timestamp with time zone,
    street varchar(250),
    city varchar(100),
    crime_type varchar(100),
    description text,
    case_number varchar(50),
    original_text text NOT NULL
);

-- Domain Driven Design (PascalCase)
CREATE TABLE "Justice"."CrimeReport" (
    "CrimeId" bigserial PRIMARY KEY,
    "FirstDate" timestamp with time zone,
    "SecondDate" timestamp with time zone,
    "Street" varchar(250),
    "City" varchar(100),
    "CrimeType" varchar(100),
    "Description" text,
    "CaseNumber" varchar(50),
    "OriginalText" text NOT NULL
);
```

Load the text into the table:

```sql
-- PostgreSQL Standard (snake_case)
COPY crime_reports (original_text)
FROM 'C:\YourDirectory\crime_reports.csv'
WITH (FORMAT CSV, HEADER OFF, QUOTE '"');

-- Domain Driven Design (PascalCase)
COPY "Justice"."CrimeReport" ("OriginalText")
FROM 'C:\YourDirectory\crime_reports.csv'
WITH (FORMAT CSV, HEADER OFF, QUOTE '"');
```

### Matching Crime Report Date Patterns

The `regexp_match()` function extracts patterns from text and returns results as an array:

```sql
-- PostgreSQL Standard (snake_case)
SELECT crime_id,
       regexp_match(original_text, '\d{1,2}\/\d{1,2}\/\d{2}')
FROM crime_reports;

-- Domain Driven Design (PascalCase)
SELECT "CrimeId",
       regexp_match("OriginalText", '\d{1,2}\/\d{1,2}\/\d{2}')
FROM "Justice"."CrimeReport";
```

Output:
```
crime_id | regexp_match
---------+-------------
       1 | {4/16/17}
       2 | {4/8/17}
       3 | {4/4/17}
       4 | {04/10/17}
       5 | {04/09/17}
```

To find all dates in each report, use `regexp_matches()` with the 'g' flag:

```sql
-- PostgreSQL Standard (snake_case)
SELECT crime_id,
       regexp_matches(original_text, '\d{1,2}\/\d{1,2}\/\d{2}', 'g')
FROM crime_reports;

-- Domain Driven Design (PascalCase)
SELECT "CrimeId",
       regexp_matches("OriginalText", '\d{1,2}\/\d{1,2}\/\d{2}', 'g')
FROM "Justice"."CrimeReport";
```

Output:
```
crime_id | regexp_matches
---------+---------------
       1 | {4/16/17}
       1 | {4/17/17}
       2 | {4/8/17}
       3 | {4/4/17}
       4 | {04/10/17}
       5 | {04/09/17}
```

To extract just the second date (when present), we can use a more specific pattern with a capture group:

```sql
-- PostgreSQL Standard (snake_case)
SELECT crime_id,
       regexp_match(original_text, '-(\d{1,2}\/\d{1,2}\/\d{2})')
FROM crime_reports;

-- Domain Driven Design (PascalCase)
SELECT "CrimeId",
       regexp_match("OriginalText", '-(\d{1,2}\/\d{1,2}\/\d{2})')
FROM "Justice"."CrimeReport";
```

Output:
```
crime_id | regexp_match
---------+-------------
       1 | {4/17/17}
       2 | 
       3 | 
       4 | 
       5 | 
```

### Matching Additional Crime Report Elements

Using specific patterns, we can extract more data elements:

```sql
-- PostgreSQL Standard (snake_case)
SELECT
    regexp_match(original_text, '(?:C0|SO)[0-9]+') AS case_number,
    regexp_match(original_text, '\d{1,2}\/\d{1,2}\/\d{2}') AS date_1,
    regexp_match(original_text, '\n(?:\w+ \w+|\w+)\n(.*):') AS crime_type,
    regexp_match(original_text, '(?:Sq.|Plz.|Dr.|Ter.|Rd.)\n(\w+ \w+|\w+)\n') AS city
FROM crime_reports;

-- Domain Driven Design (PascalCase)
SELECT
    regexp_match("OriginalText", '(?:C0|SO)[0-9]+') AS "CaseNumber",
    regexp_match("OriginalText", '\d{1,2}\/\d{1,2}\/\d{2}') AS "FirstDate",
    regexp_match("OriginalText", '\n(?:\w+ \w+|\w+)\n(.*):') AS "CrimeType",
    regexp_match("OriginalText", '(?:Sq.|Plz.|Dr.|Ter.|Rd.)\n(\w+ \w+|\w+)\n') AS "City"
FROM "Justice"."CrimeReport";
```

Output:
```
case_number    | date_1    | crime_type           | city
---------------+-----------+----------------------+-----------
{C0170006614}  | {4/16/17} | {Larceny}           | {Sterling}
{C0170006162}  | {4/8/17}  | {Destruction of...} | {Sterling}
{C0170006079}  | {4/4/17}  | {Larceny}           | {Sterling}
{SO170006250}  | {04/10/17}| {Larceny}           | {Middleburg}
{SO170006211}  | {04/09/17}| {Destruction of...} | {Sterling}
```

### Extracting Text from the regexp_match() Result

Since `regexp_match()` returns an array, we need to extract the value using array notation:

```sql
-- PostgreSQL Standard (snake_case)
SELECT
    crime_id,
    (regexp_match(original_text, '(?:C0|SO)[0-9]+'))[1] AS case_number
FROM crime_reports;

-- Domain Driven Design (PascalCase)
SELECT
    "CrimeId",
    (regexp_match("OriginalText", '(?:C0|SO)[0-9]+'))[1] AS "CaseNumber"
FROM "Justice"."CrimeReport";
```

Output:
```
crime_id | case_number
---------+------------
       1 | C0170006614
       2 | C0170006162
       3 | C0170006079
       4 | SO170006250
       5 | SO170006211
```

### Updating the crime_reports Table with Extracted Data

To update the first date field with timestamp data:

```sql
-- PostgreSQL Standard (snake_case)
UPDATE crime_reports
SET date_1 =
(
    (regexp_match(original_text, '\d{1,2}\/\d{1,2}\/\d{2}'))[1]
    || ' ' ||
    (regexp_match(original_text, '\/\d{2}\n(\d{4})'))[1]
    || ' US/Eastern'
)::timestamptz;

-- Domain Driven Design (PascalCase)
UPDATE "Justice"."CrimeReport"
SET "FirstDate" =
(
    (regexp_match("OriginalText", '\d{1,2}\/\d{1,2}\/\d{2}'))[1]
    || ' ' ||
    (regexp_match("OriginalText", '\/\d{2}\n(\d{4})'))[1]
    || ' US/Eastern'
)::timestamptz;
```

The full update for all columns, handling special cases with CASE statements:

```sql
-- PostgreSQL Standard (snake_case)
UPDATE crime_reports
SET date_1 =
(
    (regexp_match(original_text, '\d{1,2}\/\d{1,2}\/\d{2}'))[1]
    || ' ' ||
    (regexp_match(original_text, '\/\d{2}\n(\d{4})'))[1]
    || ' US/Eastern'
)::timestamptz,
date_2 =
CASE
    WHEN (SELECT regexp_match(original_text, '-(\d{1,2}\/\d{1,2}\/\d{1,2})') IS NULL)
         AND (SELECT regexp_match(original_text, '\/\d{2}\n\d{4}-(\d{4})') IS NOT NULL)
    THEN
        ((regexp_match(original_text, '\d{1,2}\/\d{1,2}\/\d{2}'))[1]
        || ' ' ||
        (regexp_match(original_text, '\/\d{2}\n\d{4}-(\d{4})'))[1]
        || ' US/Eastern'
        )::timestamptz
    WHEN (SELECT regexp_match(original_text, '-(\d{1,2}\/\d{1,2}\/\d{1,2})') IS NOT NULL)
         AND (SELECT regexp_match(original_text, '\/\d{2}\n\d{4}-(\d{4})') IS NOT NULL)
    THEN
        ((regexp_match(original_text, '-(\d{1,2}\/\d{1,2}\/\d{1,2})'))[1]
        || ' ' ||
        (regexp_match(original_text, '\/\d{2}\n\d{4}-(\d{4})'))[1]
        || ' US/Eastern'
        )::timestamptz
    ELSE NULL
END,
street = (regexp_match(original_text, 'hrs.\n(\d+ .+(?:Sq.|Plz.|Dr.|Ter.|Rd.))'))[1],
city = (regexp_match(original_text, '(?:Sq.|Plz.|Dr.|Ter.|Rd.)\n(\w+ \w+|\w+)\n'))[1],
crime_type = (regexp_match(original_text, '\n(?:\w+ \w+|\w+)\n(.*):'))[1],
description = (regexp_match(original_text, ':\s(.+)(?:C0|SO)'))[1],
case_number = (regexp_match(original_text, '(?:C0|SO)[0-9]+'))[1];

-- Domain Driven Design (PascalCase)
UPDATE "Justice"."CrimeReport"
SET "FirstDate" =
(
    (regexp_match("OriginalText", '\d{1,2}\/\d{1,2}\/\d{2}'))[1]
    || ' ' ||
    (regexp_match("OriginalText", '\/\d{2}\n(\d{4})'))[1]
    || ' US/Eastern'
)::timestamptz,
"SecondDate" =
CASE
    WHEN (SELECT regexp_match("OriginalText", '-(\d{1,2}\/\d{1,2}\/\d{1,2})') IS NULL)
         AND (SELECT regexp_match("OriginalText", '\/\d{2}\n\d{4}-(\d{4})') IS NOT NULL)
    THEN
        ((regexp_match("OriginalText", '\d{1,2}\/\d{1,2}\/\d{2}'))[1]
        || ' ' ||
        (regexp_match("OriginalText", '\/\d{2}\n\d{4}-(\d{4})'))[1]
        || ' US/Eastern'
        )::timestamptz
    WHEN (SELECT regexp_match("OriginalText", '-(\d{1,2}\/\d{1,2}\/\d{1,2})') IS NOT NULL)
         AND (SELECT regexp_match("OriginalText", '\/\d{2}\n\d{4}-(\d{4})') IS NOT NULL)
    THEN
        ((regexp_match("OriginalText", '-(\d{1,2}\/\d{1,2}\/\d{1,2})'))[1]
        || ' ' ||
        (regexp_match("OriginalText", '\/\d{2}\n\d{4}-(\d{4})'))[1]
        || ' US/Eastern'
        )::timestamptz
    ELSE NULL
END,
"Street" = (regexp_match("OriginalText", 'hrs.\n(\d+ .+(?:Sq.|Plz.|Dr.|Ter.|Rd.))'))[1],
"City" = (regexp_match("OriginalText", '(?:Sq.|Plz.|Dr.|Ter.|Rd.)\n(\w+ \w+|\w+)\n'))[1],
"CrimeType" = (regexp_match("OriginalText", '\n(?:\w+ \w+|\w+)\n(.*):'))[1],
"Description" = (regexp_match("OriginalText", ':\s(.+)(?:C0|SO)'))[1],
"CaseNumber" = (regexp_match("OriginalText", '(?:C0|SO)[0-9]+'))[1];
```

After updating, we can query the structured data:

```sql
-- PostgreSQL Standard (snake_case)
SELECT date_1, street, city, crime_type
FROM crime_reports;

-- Domain Driven Design (PascalCase)
SELECT "FirstDate", "Street", "City", "CrimeType"
FROM "Justice"."CrimeReport";
```

Output:
```
date_1                  | street                     | city       | crime_type
------------------------+----------------------------+------------+-----------------
2017-04-16 21:00:00-04 | 46000 Block Ashmere Sq.    | Sterling   | Larceny
2017-04-08 16:00:00-04 | 46000 Block Potomac Run Plz.| Sterling   | Destruction of...
2017-04-04 14:00:00-04 | 24000 Block Hawthorn...    | Sterling   | Larceny
2017-04-10 16:05:00-04 | 21800 block Newlin Mill Rd.| Middleburg | Larceny
2017-04-09 12:00:00-04 | 470000 block Fairway Dr.   | Sterling   | Destruction of...
```

## Using Regular Expressions with WHERE

PostgreSQL allows using regular expressions in WHERE clauses:

```sql
-- PostgreSQL Standard (snake_case)
SELECT geo_name
FROM us_counties_2010
WHERE geo_name ~* '(.+lade.+|.+lare.+)'
ORDER BY geo_name;

SELECT geo_name
FROM us_counties_2010
WHERE geo_name ~* '.+ash.+' AND geo_name !~ 'Wash.+'
ORDER BY geo_name;

-- Domain Driven Design (PascalCase)
SELECT "GeoName"
FROM "Geography"."USCounty2010"
WHERE "GeoName" ~* '(.+lade.+|.+lare.+)'
ORDER BY "GeoName";

SELECT "GeoName"
FROM "Geography"."USCounty2010"
WHERE "GeoName" ~* '.+ash.+' AND "GeoName" !~ 'Wash.+'
ORDER BY "GeoName";
```

Regular expression operators:
- `~` performs case-sensitive matching
- `~*` performs case-insensitive matching
- `!~` negates a case-sensitive match
- `!~*` negates a case-insensitive match

## Additional Regular Expression Functions

```sql
-- PostgreSQL Standard (snake_case)
SELECT regexp_replace('05/12/2018', '\d{4}', '2017');
-- Returns: 05/12/2017

SELECT regexp_split_to_table('Four,score,and,seven,years,ago', ',');
-- Returns rows: Four, score, and, seven, years, ago

SELECT regexp_split_to_array('Phil Mike Tony Steve', ' ');
-- Returns: {Phil,Mike,Tony,Steve}

SELECT array_length(regexp_split_to_array('Phil Mike Tony Steve', ' '), 1);
-- Returns: 4

-- Domain Driven Design (PascalCase)
SELECT regexp_replace('05/12/2018', '\d{4}', '2017') FROM "Text"."Sample";
-- Returns: 05/12/2017

SELECT regexp_split_to_table('Four,score,and,seven,years,ago', ',') FROM "Text"."Sample";
-- Returns rows: Four, score, and, seven, years, ago

SELECT regexp_split_to_array('Phil Mike Tony Steve', ' ') FROM "Text"."Sample";
-- Returns: {Phil,Mike,Tony,Steve}

SELECT array_length(regexp_split_to_array('Phil Mike Tony Steve', ' '), 1) FROM "Text"."Sample";
-- Returns: 4
```

These functions provide additional text manipulation capabilities:
- `regexp_replace()` substitutes text matching a pattern
- `regexp_split_to_table()` splits text into rows
- `regexp_split_to_array()` splits text into an array
- `array_length()` counts elements in an array

## Full Text Search in PostgreSQL

PostgreSQL's full-text search provides advanced capabilities for searching large amounts of text.

### Text Search Data Types

PostgreSQL's text search implementation includes two data types:
- `tsvector`: Represents optimized text to be searched
- `tsquery`: Represents search query terms and operators

#### Storing Text as Lexemes with tsvector

```sql
-- PostgreSQL Standard (snake_case)
SELECT to_tsvector('I am walking across the sitting room to sit with you.');
-- Returns: 'across':4 'room':7 'sit':9 'walk':3

-- Domain Driven Design (PascalCase)
SELECT to_tsvector('I am walking across the sitting room to sit with you.') FROM "Text"."Sample";
-- Returns: 'across':4 'room':7 'sit':9 'walk':3
```

The `to_tsvector()` function reduces text to lexemes (word stems) and removes stop words.

#### Creating the Search Terms with tsquery

```sql
-- PostgreSQL Standard (snake_case)
SELECT to_tsquery('walking & sitting');
-- Returns: 'walk' & 'sit'

-- Domain Driven Design (PascalCase)
SELECT to_tsquery('walking & sitting') FROM "Text"."Sample";
-- Returns: 'walk' & 'sit'
```

The `to_tsquery()` function converts search terms to the `tsquery` format.

### Using the @@ Match Operator for Searching

```sql
-- PostgreSQL Standard (snake_case)
SELECT to_tsvector('I am walking across the sitting room') @@ to_tsquery('walking & sitting');
-- Returns: true

SELECT to_tsvector('I am walking across the sitting room') @@ to_tsquery('walking & running');
-- Returns: false

-- Domain Driven Design (PascalCase)
SELECT to_tsvector('I am walking across the sitting room') @@ to_tsquery('walking & sitting') 
FROM "Text"."Sample";
-- Returns: true

SELECT to_tsvector('I am walking across the sitting room') @@ to_tsquery('walking & running') 
FROM "Text"."Sample";
-- Returns: false
```

The `@@` operator tests whether a `tsvector` matches a `tsquery`.

### Creating a Table for Full Text Search

```sql
-- PostgreSQL Standard (snake_case)
CREATE TABLE president_speeches (
    sotu_id serial PRIMARY KEY,
    president varchar(100) NOT NULL,
    title varchar(250) NOT NULL,
    speech_date date NOT NULL,
    speech_text text NOT NULL,
    search_speech_text tsvector
);

-- Domain Driven Design (PascalCase)
CREATE TABLE "Politics"."PresidentialSpeech" (
    "SotuId" serial PRIMARY KEY,
    "President" varchar(100) NOT NULL,
    "Title" varchar(250) NOT NULL,
    "SpeechDate" date NOT NULL,
    "SpeechText" text NOT NULL,
    "SearchSpeechText" tsvector
);
```

Fill the table with speech data:

```sql
-- PostgreSQL Standard (snake_case)
COPY president_speeches (president, title, speech_date, speech_text)
FROM 'C:\YourDirectory\sotu-1946-1977.csv'
WITH (FORMAT CSV, DELIMITER '|', HEADER OFF, QUOTE '"');

-- Domain Driven Design (PascalCase)
COPY "Politics"."PresidentialSpeech" ("President", "Title", "SpeechDate", "SpeechText")
FROM 'C:\YourDirectory\sotu-1946-1977.csv'
WITH (FORMAT CSV, DELIMITER '|', HEADER OFF, QUOTE '"');
```

Convert the speech text to tsvector format:

```sql
-- PostgreSQL Standard (snake_case)
UPDATE president_speeches
SET search_speech_text = to_tsvector('english', speech_text);

-- Domain Driven Design (PascalCase)
UPDATE "Politics"."PresidentialSpeech"
SET "SearchSpeechText" = to_tsvector('english', "SpeechText");
```

Create a GIN (Generalized Inverted Index) to speed up searches:

```sql
-- PostgreSQL Standard (snake_case)
CREATE INDEX search_idx ON president_speeches USING gin(search_speech_text);

-- Domain Driven Design (PascalCase)
CREATE INDEX "IX_Politics_PresidentialSpeech_SearchSpeechText" 
ON "Politics"."PresidentialSpeech" 
USING gin("SearchSpeechText");
```

### Searching Speech Text

Find speeches mentioning Vietnam:

```sql
-- PostgreSQL Standard (snake_case)
SELECT president, speech_date
FROM president_speeches
WHERE search_speech_text @@ to_tsquery('Vietnam')
ORDER BY speech_date;

-- Domain Driven Design (PascalCase)
SELECT "President", "SpeechDate"
FROM "Politics"."PresidentialSpeech"
WHERE "SearchSpeechText" @@ to_tsquery('Vietnam')
ORDER BY "SpeechDate";
```

### Showing Search Result Locations

The `ts_headline()` function highlights search terms in their context:

```sql
-- PostgreSQL Standard (snake_case)
SELECT president,
       speech_date,
       ts_headline(speech_text, to_tsquery('Vietnam'),
                  'StartSel = <,
                   StopSel = >,
                   MinWords=5,
                   MaxWords=7,
                   MaxFragments=1')
FROM president_speeches
WHERE search_speech_text @@ to_tsquery('Vietnam');

-- Domain Driven Design (PascalCase)
SELECT "President",
       "SpeechDate",
       ts_headline("SpeechText", to_tsquery('Vietnam'),
                  'StartSel = <,
                   StopSel = >,
                   MinWords=5,
                   MaxWords=7,
                   MaxFragments=1')
FROM "Politics"."PresidentialSpeech"
WHERE "SearchSpeechText" @@ to_tsquery('Vietnam');
```

### Using Multiple Search Terms

Find speeches mentioning transportation but not roads:

```sql
-- PostgreSQL Standard (snake_case)
SELECT president,
       speech_date,
       ts_headline(speech_text, to_tsquery('transportation & !roads'),
                  'StartSel = <,
                   StopSel = >,
                   MinWords=5,
                   MaxWords=7,
                   MaxFragments=1')
FROM president_speeches
WHERE search_speech_text @@ to_tsquery('transportation & !roads');

### Using Multiple Search Terms (continued)

```sql
-- PostgreSQL Standard (snake_case)
SELECT president,
       speech_date,
       ts_headline(speech_text, to_tsquery('transportation & !roads'),
                  'StartSel = <,
                   StopSel = >,
                   MinWords=5,
                   MaxWords=7,
                   MaxFragments=1')
FROM president_speeches
WHERE search_speech_text @@ to_tsquery('transportation & !roads');

-- Domain Driven Design (PascalCase)
SELECT "President",
       "SpeechDate",
       ts_headline("SpeechText", to_tsquery('transportation & !roads'),
                  'StartSel = <,
                   StopSel = >,
                   MinWords=5,
                   MaxWords=7,
                   MaxFragments=1')
FROM "Politics"."PresidentialSpeech"
WHERE "SearchSpeechText" @@ to_tsquery('transportation & !roads');
```

Output:

```markdown
president         | speech_date | ts_headline
------------------+-------------+---------------------------------------
Harry S. Truman   | 1947-01-06  | such industries as <transportation>, coal, oil, steel
Harry S. Truman   | 1949-01-05  | field of <transportation>
John F. Kennedy   | 1961-01-30  | Obtaining additional air <transport> mobility--and obtaining
Lyndon B. Johnson | 1964-01-08  | reformed our tangled <transportation> and transit policies
```

Notice that the highlighted words in the ts_headline column include "transportation" and "transport". This occurs because the to_tsquery() function converts words to their lexemes (stems) for searching, allowing matches with related words.

### Searching for Adjacent Words

The `<->` operator finds adjacent words:

```sql
-- PostgreSQL Standard (snake_case)
SELECT president,
       speech_date,
       ts_headline(speech_text, to_tsquery('military <-> defense'),
                  'StartSel = <,
                   StopSel = >,
                   MinWords=5,
                   MaxWords=7,
                   MaxFragments=1')
FROM president_speeches
WHERE search_speech_text @@ to_tsquery('military <-> defense');

-- Domain Driven Design (PascalCase)
SELECT "President",
       "SpeechDate",
       ts_headline("SpeechText", to_tsquery('military <-> defense'),
                  'StartSel = <,
                   StopSel = >,
                   MinWords=5,
                   MaxWords=7,
                   MaxFragments=1')
FROM "Politics"."PresidentialSpeech"
WHERE "SearchSpeechText" @@ to_tsquery('military <-> defense');
```

Output:

```markdown
president             | speech_date | ts_headline
----------------------+-------------+-----------------------------------------
Dwight D. Eisenhower  | 1956-01-05  | system our <military> <defenses> are designed
Dwight D. Eisenhower  | 1958-01-09  | direct <military> <defense> efforts, but likewise
Dwight D. Eisenhower  | 1959-01-09  | survival--the <military> <defense> of national life
Richard M. Nixon      | 1972-01-20  | spending. Strong <military> <defenses>
```

The `<->` operator finds the word "military" immediately followed by "defense" or "defenses" (as lexemes). The syntax can be modified to search for words that are a specific distance apart by adding a number between the symbols: `<2>` would find words that are two words apart.

## Ranking Query Matches by Relevance

PostgreSQL offers two functions to rank search results by relevance:

- `ts_rank()`: Ranks based on how often search terms appear
- `ts_rank_cd()`: Considers how close search terms are to each other (cover density)

### Basic Ranking with ts_rank()

```sql
-- PostgreSQL Standard (snake_case)
SELECT president,
       speech_date,
       ts_rank(search_speech_text,
               to_tsquery('war & security & threat & enemy')) AS score
FROM president_speeches
WHERE search_speech_text @@ to_tsquery('war & security & threat & enemy')
ORDER BY score DESC
LIMIT 5;

-- Domain Driven Design (PascalCase)
SELECT "President",
       "SpeechDate",
       ts_rank("SearchSpeechText",
               to_tsquery('war & security & threat & enemy')) AS "Score"
FROM "Politics"."PresidentialSpeech"
WHERE "SearchSpeechText" @@ to_tsquery('war & security & threat & enemy')
ORDER BY "Score" DESC
LIMIT 5;
```

Output:

```markdown
President           | SpeechDate  | Score
--------------------+-------------+-----------
Harry S. Truman     | 1946-01-21  | 0.257522
Lyndon B. Johnson   | 1968-01-17  | 0.186296
Dwight D. Eisenhower| 1957-01-10  | 0.140851
Harry S. Truman     | 1952-01-09  | 0.0982469
Richard M. Nixon    | 1972-01-20  | 0.0973585
```

The scores are arbitrary values useful for sorting but don't have inherent meaning across different queries.

### Normalized Ranking with ts_rank()

We can normalize the ranking by speech length by adding a normalization code:

```sql
-- PostgreSQL Standard (snake_case)
SELECT president,
       speech_date,
       ts_rank(search_speech_text,
               to_tsquery('war & security & threat & enemy'), 2)::numeric AS score
FROM president_speeches
WHERE search_speech_text @@ to_tsquery('war & security & threat & enemy')
ORDER BY score DESC
LIMIT 5;

-- Domain Driven Design (PascalCase)
SELECT "President",
       "SpeechDate",
       ts_rank("SearchSpeechText",
               to_tsquery('war & security & threat & enemy'), 2)::numeric AS "Score"
FROM "Politics"."PresidentialSpeech"
WHERE "SearchSpeechText" @@ to_tsquery('war & security & threat & enemy')
ORDER BY "Score" DESC
LIMIT 5;
```

Output:

```markdown
President           | SpeechDate  | Score
--------------------+-------------+---------------
Lyndon B. Johnson   | 1968-01-17  | 0.0000728288
Dwight D. Eisenhower| 1957-01-10  | 0.0000633609
Richard M. Nixon    | 1972-01-20  | 0.0000497998
Harry S. Truman     | 1952-01-09  | 0.0000365366
Dwight D. Eisenhower| 1958-01-09  | 0.0000355315
```

The normalization code `2` divides the score by the document length, giving a more accurate comparison between speeches of different lengths.

## Practical Applications and Key Takeaways

Text mining techniques provide powerful tools to extract structured data from unstructured text. These techniques are valuable when:

1. **Transforming Text Reports**: Converting unstructured reports (like crime incidents or public speeches) into structured data for analysis.

2. **Information Extraction**: Identifying and extracting specific entities like dates, locations, and numbers from large volumes of text.

3. **Pattern Identification**: Finding recurring patterns or themes in text data.

4. **Content Relevance**: Ranking text content based on relevance to specific search terms.

5. **Corpus Analysis**: Analyzing large collections of documents to identify trends, sentiments, or topic frequencies.

The process typically involves:

1. **Text Normalization**: Using string functions to standardize text format.
2. **Pattern Matching**: Using regular expressions to identify specific patterns.
3. **Information Extraction**: Extracting the identified patterns into structured data.
4. **Search and Analysis**: Using full-text search capabilities to find and rank relevant content.

This approach allows analysts to derive meaning from previously unstructured text data, opening up new sources of information for data analysis.

## Try It Yourself

1. The style guide of a publishing company wants you to avoid commas before suffixes in names. Find a solution to remove commas in names like "Alvarez, Jr." and "Williams, Sr." and consider how to place suffixes in a separate column.

2. Using any State of the Union address, count unique words that are five characters or more. Consider using `regexp_split_to_table()` in a subquery to create a table of words to count. Bonus: Remove commas and periods at the end of each word.

3. Rewrite the query in Listing 13-25 using `ts_rank_cd()` instead of `ts_rank()` and compare the results. Does using the `ts_rank_cd()` function significantly change the ranking order?

## Summary

This chapter explored techniques for extracting meaningful data from text:

- **String Functions**: Basic text transformation functions like `upper()`, `lower()`, `trim()`, and `replace()` help normalize and standardize text.

- **Regular Expressions**: Pattern matching using regular expressions enables extraction of specific data elements from text, turning unstructured text into structured data.

- **Full-Text Search**: PostgreSQL's full-text search capabilities allow for powerful search and ranking of text content.

PostgreSQL offers powerful tools for text analysis that can turn unstructured text into structured data for analysis. Whether working with crime reports, speeches, or any text-based content, these techniques help reveal insights that would otherwise remain hidden within the text.

The value of these text mining approaches extends beyond the examples shown. By building your own datasets from text sources, you create unique analytical opportunities and can continuously update and mine this data for trends and patterns over time.