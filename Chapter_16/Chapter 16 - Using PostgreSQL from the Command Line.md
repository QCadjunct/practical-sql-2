# Chapter 16 - Using PostgreSQL from the Command Line

## Introduction

This chapter introduces the command line interface for PostgreSQL, focusing on the `psql` tool. Before graphical user interfaces (GUIs) became standard, command line interfaces were the primary way to interact with computers. Even today, using the command line offers several advantages:

- Faster execution through short commands instead of navigating through GUI menu layers
- Access to functions only available via command line
- Ability to work in environments where only command line access is available

The chapter covers setting up and accessing your computer's command line, launching `psql`, and using it to run queries, manage database objects, and interact with your operating system through text commands.

## Setting Up the Command Line for psql

### Environment Variables

The first step is setting up an environment variable called `path` that tells your system where to find `psql`. Environment variables store parameters that specify system or application configurations. The `path` variable stores directory names containing executable programs, allowing you to run `psql` without entering its full directory path.

### Windows psql Setup

#### Adding psql to the Windows PATH

For Windows users, the process involves:

1. Opening Windows Control Panel
2. Accessing Environment Variables
3. Adding PostgreSQL's bin directory to your PATH (e.g., `C:\Program Files\PostgreSQL\10\bin`)

#### Launching and Configuring Windows Command Prompt

**PostgreSQL Standard (snake_case):**
```
C:\Users\Username>
```

**Domain Driven Design (PascalCase):**
```
C:\Users\Username>
```

Windows Command Prompt can be customized for better output display by:
- Setting window size to width of 80 and height of 25
- Using Lucida Console 14 or another preferred font

#### Windows Command Line Instructions

Useful Windows commands include:

| Command | Function | Example | Action |
|---------|----------|---------|--------|
| cd | Change directory | `cd C:\my-stuff` | Change to the my-stuff directory |
| copy | Copy a file | `copy C:\my-stuff\song.mp3 C:\Music\song_favorite.mp3` | Copy a file with new name |
| del | Delete | `del *.jpg` | Delete all JPG files in current directory |
| dir | List directory contents | `dir /p` | Show directory contents one screen at a time |
| findstr | Find string matches | `findstr "peach" *.txt` | Search for "peach" in all text files |
| mkdir | Make a directory | `mkdir C:\my-stuff\Salad` | Create a new directory |
| move | Move a file | `move C:\my-stuff\song.mp3 C:\Music\` | Move file to new location |

### macOS psql Setup

#### Adding psql to the macOS PATH

On macOS, the bash shell is used through Terminal. To add `psql` to the PATH:

1. Show hidden files in Finder
2. Edit `.bash_profile` in your home directory
3. Add the following line:
   ```
   export PATH="/Applications/Postgres.app/Contents/Versions/latest/bin:$PATH"
   ```

#### Launching and Configuring macOS Terminal

**PostgreSQL Standard (snake_case):**
```
username@machinename:~ $
```

**Domain Driven Design (PascalCase):**
```
username@machinename:~ $
```

The Terminal interface can be customized by:
- Setting window size to 80 columns by 25 rows
- Choosing Monaco 14 or another preferred font

#### macOS Terminal Commands

Useful Terminal commands include:

| Command | Function | Example | Action |
|---------|----------|---------|--------|
| cd | Change directory | `cd /Users/pparker/my-stuff/` | Change to the my-stuff directory |
| cp | Copy files | `cp song.mp3 song_backup.mp3` | Create a backup copy of a file |
| grep | Find strings | `grep 'us_counties_2010' *.sql` | Find pattern matches in files |
| ls | List directory contents | `ls -al` | List all files in long format |
| mkdir | Make a directory | `mkdir resumes` | Create a new directory |
| mv | Move a file | `mv song.mp3 /Users/pparker/songs` | Move a file to another location |
| rm | Remove files | `rm *.jpg` | Delete all JPG files |

### Linux psql Setup

On Linux, `psql` is generally added to the PATH automatically during installation. Users can launch a terminal (often using Ctrl+Alt+T) and begin using the commands that are similar to those in macOS.

## Working with psql

### Launching psql and Connecting to a Database

To launch `psql` and connect to a database, the pattern is:

**PostgreSQL Standard (snake_case):**
```
psql -d database_name -U user_name
```

**Domain Driven Design (PascalCase):**
```
psql -d "DatabaseName" -U "UserName"
```

For example, to connect to the `analysis` database with the `postgres` user:

**PostgreSQL Standard (snake_case):**
```
psql -d analysis -U postgres
```

**Domain Driven Design (PascalCase):**
```
psql -d "Analysis" -U "Postgres"
```

For remote connections, add the host parameter:

**PostgreSQL Standard (snake_case):**
```
psql -d analysis -U postgres -h example.com
```

**Domain Driven Design (PascalCase):**
```
psql -d "Analysis" -U "Postgres" -h example.com
```

After successfully connecting, you'll see a prompt like:

**PostgreSQL Standard (snake_case):**
```
psql (10.1)
Type "help" for help.
analysis=#
```

**Domain Driven Design (PascalCase):**
```
psql (10.1)
Type "help" for help.
Analysis=#
```

The `#` symbol indicates you're connected with superuser privileges. Non-superusers will see a `>` symbol instead.

### Getting Help

Several help commands are available within `psql`:

| Command | Displays |
|---------|----------|
| \? | Commands available within psql |
| \? options | Options for use with psql |
| \? variables | Variables for use with psql |
| \h | List of SQL commands (add command name for detailed help) |

### Changing the User and Database Connection

The `\c` meta-command lets you connect to a different database or switch user accounts:

**PostgreSQL Standard (snake_case):**
```
analysis=# \c gis_analysis
You are now connected to database "gis_analysis" as user "postgres".
```

**Domain Driven Design (PascalCase):**
```
Analysis=# \c "GisAnalysis"
You are now connected to database "GisAnalysis" as user "Postgres".
```

To switch both database and user:

**PostgreSQL Standard (snake_case):**
```
analysis=# \c gis_analysis anthony
You are now connected to database "gis_analysis" as user "anthony".
```

**Domain Driven Design (PascalCase):**
```
Analysis=# \c "GisAnalysis" "Anthony"
You are now connected to database "GisAnalysis" as user "Anthony".
```

## Running SQL Queries in psql

### Single-line Queries

**PostgreSQL Standard (snake_case):**
```sql
analysis=# SELECT geo_name FROM us_counties_2010 LIMIT 3;
geo_name
--------------
Autauga County
Baldwin County
Barbour County
(3 rows)
```

**Domain Driven Design (PascalCase):**
```sql
Analysis=# SELECT "GeoName" FROM "Geography"."County" LIMIT 3;
GeoName
--------------
Autauga County
Baldwin County
Barbour County
(3 rows)
```

### Multi-line Queries

**PostgreSQL Standard (snake_case):**
```sql
analysis=# SELECT geo_name
analysis-# FROM us_counties_2010
analysis-# LIMIT 3;
```

**Domain Driven Design (PascalCase):**
```sql
Analysis=# SELECT "GeoName"
Analysis-# FROM "Geography"."County"
Analysis-# LIMIT 3;
```

Note how the prompt changes from `=#` to `-#` when entering a multi-line query.

### Checking for Open Parentheses

The `psql` prompt shows when you haven't closed a pair of parentheses:

**PostgreSQL Standard (snake_case):**
```sql
analysis=# CREATE TABLE wineries (
analysis(# id bigint,
analysis(# winery_name varchar(100)
analysis(# );
CREATE TABLE
```

**Domain Driven Design (PascalCase):**
```sql
Analysis=# CREATE TABLE "Winery"."Winery" (
Analysis(# "Id" bigint,
Analysis(# "Name" varchar(100)
Analysis(# );
CREATE TABLE
```

The prompt changes to include an open parenthesis, helping you keep track of syntax.

### Editing Queries

To modify a query in `psql`, use `\e` or `\edit` which will open the last executed query in a text editor:
- Windows: Opens in Notepad
- macOS/Linux: Opens in vim (press `i` to edit, `ESC` then `:wq` to save and quit)

## Navigating and Formatting Results

### Setting Paging of Results

For queries with many results, `psql` pages the output automatically. Press Q to exit the paging view. You can toggle paging with:

```
\pset pager
```

### Formatting the Results Grid

You can customize output display with `\pset` options:

- `\pset border int` - Sets border style (0, 1, or 2)
- `\pset format unaligned` - Displays results as delimited text
- `\pset fieldsep ','` - Sets field separator (comma in this example)
- `\pset footer` - Toggles display of result row count
- `\pset null 'null'` - Sets how null values display

### Viewing Expanded Results

For better viewing of wide data, toggle expanded display with `\x`:

**PostgreSQL Standard (snake_case):**
```sql
analysis=# \x
Expanded display is on.
analysis=# SELECT * FROM grades;
-[ RECORD 1 ]------------------
student_id | 1
course_id  | 2
course     | English 11B
grade      | D
-[ RECORD 2 ]------------------
student_id | 1
course_id  | 3
course     | World History 11B
grade      | C
```

**Domain Driven Design (PascalCase):**
```sql
Analysis=# \x
Expanded display is on.
Analysis=# SELECT * FROM "Education"."Grade";
-[ RECORD 1 ]------------------
StudentId | 1
CourseId  | 2
Course    | English 11B
Grade     | D
-[ RECORD 2 ]------------------
StudentId | 1
CourseId  | 3
Course    | World History 11B
Grade     | C
```

You can also set `\x auto` to let PostgreSQL automatically choose the display format based on output size.

## Meta-Commands for Database Information

The `\d` series of commands provides information about database objects:

| Command | Displays |
|---------|----------|
| \d [pattern] | Columns, data types, and other information on objects |
| \di [pattern] | Indexes and their associated tables |
| \dt [pattern] | Tables and their owners |
| \du [pattern] | User accounts and attributes |
| \dv [pattern] | Views and their owners |
| \dx [pattern] | Installed extensions |

Example of `\dt+` output:

**PostgreSQL Standard (snake_case):**
```
List of relations
Schema | Name              | Type  | Owner    | Size   | Description
-------+-------------------+-------+----------+--------+-------------
public | acs_2011_2015_stats | table | postgres | 320 kB |
public | crime_reports      | table | postgres | 16 kB  |
public | us_counties_2000   | table | postgres | 336 kB |
public | us_counties_2010   | table | postgres | 1352 kB |
```

**Domain Driven Design (PascalCase):**
```
List of relations
Schema     | Name          | Type  | Owner    | Size   | Description
-----------+---------------+-------+----------+--------+-------------
Census     | AcsStats      | table | Postgres | 320 kB |
Police     | CrimeReport   | table | Postgres | 16 kB  |
Geography  | County2000    | table | Postgres | 336 kB |
Geography  | County        | table | Postgres | 1352 kB |
```

You can filter the output by adding a pattern: `\dt+ us*` would show only tables starting with "us".

## Importing, Exporting, and Using Files

### Using \copy for Import and Export

The `\copy` meta-command allows importing and exporting data between the local machine and the PostgreSQL server:

**PostgreSQL Standard (snake_case):**
```sql
analysis=# DROP TABLE state_regions;
DROP TABLE
analysis=# CREATE TABLE state_regions (
analysis(# st varchar(2) CONSTRAINT st_key PRIMARY KEY,
analysis(# region varchar(20) NOT NULL
analysis(# );
CREATE TABLE
analysis=# \copy state_regions FROM 'C:\YourDirectory\state_regions.csv' WITH (FORMAT CSV, HEADER);
COPY 56
```

**Domain Driven Design (PascalCase):**
```sql
Analysis=# DROP TABLE "Geography"."StateRegion";
DROP TABLE
Analysis=# CREATE TABLE "Geography"."StateRegion" (
Analysis(# "StateCode" varchar(2) CONSTRAINT "PK_Geography_StateRegion" PRIMARY KEY,
Analysis(# "Region" varchar(20) NOT NULL
Analysis(# );
CREATE TABLE
Analysis=# \copy "Geography"."StateRegion" FROM 'C:\YourDirectory\state_regions.csv' WITH (FORMAT CSV, HEADER);
COPY 56
```

### Saving Query Output to a File

To save query output to a file, use the `\o` meta-command:

**PostgreSQL Standard (snake_case):**
```sql
analysis=# \a \f , \pset footer
Output format is unaligned.
Field separator is ",".
Default footer is off.
analysis=# SELECT * FROM grades;
student_id,course_id,course,grade
1,2,English 11B,D
1,3,World History 11B,C
1,4,Trig 2,B
1,1,Biology 2,C
analysis=# \o 'C:/YourDirectory/query_output.csv'
analysis=# SELECT * FROM grades;
analysis=#
```

**Domain Driven Design (PascalCase):**
```sql
Analysis=# \a \f , \pset footer
Output format is unaligned.
Field separator is ",".
Default footer is off.
Analysis=# SELECT * FROM "Education"."Grade";
StudentId,CourseId,Course,Grade
1,2,English 11B,D
1,3,World History 11B,C
1,4,Trig 2,B
1,1,Biology 2,C
Analysis=# \o 'C:/YourDirectory/query_output.csv'
Analysis=# SELECT * FROM "Education"."Grade";
Analysis=#
```

After specifying the output file, query results will be appended to that file until you enter `\o` with no filename to return to screen output.

### Reading and Executing SQL Stored in a File

To run SQL stored in a text file from the command line:

**PostgreSQL Standard (snake_case):**
```
psql -d analysis -U postgres -f display-grades.sql
```

**Domain Driven Design (PascalCase):**
```
psql -d "Analysis" -U "Postgres" -f display-grades.sql
```

This approach is useful for repetitive tasks and can execute multiple queries in succession.

## Additional Command Line Utilities

PostgreSQL includes several command line utilities beyond `psql`:

### Creating a Database with createdb

Instead of using SQL statements, you can create databases directly from the command line:

**PostgreSQL Standard (snake_case):**
```
createdb -U postgres -e box_office
```

**Domain Driven Design (PascalCase):**
```
createdb -U "Postgres" -e "BoxOffice"
```

The `-e` flag echoes the SQL command being executed.

### Loading Shapefiles with shp2pgsql

To import a shapefile into a PostGIS database:

**PostgreSQL Standard (snake_case):**
```
shp2pgsql -I -s 4269 -W Latin1 tl_2010_us_county10.shp us_counties_2010_shp | psql -d gis_analysis -U postgres
```

**Domain Driven Design (PascalCase):**
```
shp2pgsql -I -s 4269 -W Latin1 tl_2010_us_county10.shp "Geography"."CountyShape" | psql -d "GisAnalysis" -U "Postgres"
```

The command breaks down as follows:
- `-I` adds a GiST index on the geometry column
- `-s` specifies the SRID for geometric data
- `-W` specifies the encoding (Latin1 for census shapefiles)
- The pipe (`|`) redirects the output to the `psql` command

## Summary

The command line interface offers a powerful alternative to GUI tools for PostgreSQL database management. Key benefits include:

1. **Efficiency**: Faster execution through direct commands
2. **Access**: Ability to use functions not available in GUIs
3. **Flexibility**: Works in environments where GUI access is limited
4. **Automation**: Easily integrates with scripts and schedulers

Working with `psql` and related command line tools gives you:

- Direct SQL query execution
- Database information retrieval through meta-commands
- Data import and export capabilities
- Output formatting options
- Ability to save and execute SQL from files
- Database administration tools like `createdb` and `shp2pgsql`

Whether you're a beginner or experienced user, command line proficiency significantly enhances your PostgreSQL workflow, especially for repetitive tasks or when working in environments with limited GUI access.