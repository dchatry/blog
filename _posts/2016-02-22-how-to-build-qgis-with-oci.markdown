---
layout: post
title: Collection of useful PostgreSQL scripts and commands
date: 20:52 29-07-2015
headline: Updated regularly!
taxonomy:
    category: blog
    tag: [database]
---
A collection of useful scripts and queries to manage and analyse PostgreSQL databases.

*This article will be updated regularly.*

## List and describe all views/tables

<u>Usage</u>: sh list_tables.sh [-U username] [-h host] [-d database_name] [-s schema] [-t] [-v] 

*Use -t if you want to show tables only, use -v if you want to show views only.*

<pre>
<code>
#!/bin/bash
# List all tables/views and their columns
# Usage: sh list_tables.sh [-U username] [-h host] [-d database_name] [-s schema] [-t] [-v] [> file.txt]
# Use -t if you want to show tables only, use -v if you want to show views only.
# 
export PGPASSWORD='yourpassword'

USER="postgres"
HOST="localhost"
SCHEMA="all"
TABLES=0
VIEWS=0

while getopts "U:h:d:s:tv" opt; do
    case "$opt" in
    U)  USER=${OPTARG}
        ;;
    h)  HOST=${OPTARG}
        ;;
    d)  DATABASE=${OPTARG}
        ;;
    s)  SCHEMA=${OPTARG}
        ;;
    t)  TABLES=1
        ;;
    v)  VIEWS=1
        ;;
    \?)
      echo "Invalid option: -$OPTARG" >&2
      ;;
    esac
done

if [ "$VIEWS" -eq 0 ] || [ "$TABLES" -eq 1 ]; then
    psql \
        -X \
        -h $HOST \
        -U $USER \
        -c "SELECT table_schema, table_name FROM INFORMATION_SCHEMA.tables WHERE table_schema NOT IN ('information_schema', 'pg_catalog') ORDER BY table_name;" \
        --single-transaction \
        --set AUTOCOMMIT=off \
        --set ON_ERROR_STOP=on \
        --no-align \
        -t \
        --field-separator ' ' \
        --quiet \
        $DATABASE | 
    while read TABLE_SCHEMA TABLE_NAME; do
        if ([ "$SCHEMA" != "all" ] && [ "$TABLE_SCHEMA" = "$SCHEMA" ]) || [ "$SCHEMA" = "all" ]; then
            psql -U $USER -d $DATABASE -c "\d $TABLE_SCHEMA.$TABLE_NAME"
        fi
    done
fi

if [ $TABLES -eq 0 ] || [ $VIEWS -eq 1 ]; then
   psql \
        -X \
        -h $HOST \
        -U $USER \
        -c "SELECT table_schema, table_name FROM INFORMATION_SCHEMA.views WHERE table_schema NOT IN ('information_schema', 'pg_catalog') ORDER BY table_name;" \
        --single-transaction \
        --set AUTOCOMMIT=off \
        --set ON_ERROR_STOP=on \
        --no-align \
        -t \
        --field-separator ' ' \
        --quiet \
        $DATABASE | 
    while read TABLE_SCHEMA TABLE_NAME; do
        if ([ "$SCHEMA" != "all" ] && [ "$TABLE_SCHEMA" = "$SCHEMA" ]) || [ "$SCHEMA" = "all" ]; then
            psql -U $USER -d $DATABASE -c "\d $TABLE_SCHEMA.$TABLE_NAME"
        fi
    done
fi
</code>
</pre>

## Clean up CSV files before importing into PostgreSQL database

Here is a neat little script that will clean up CSV files (exported from an **Oracle** database) and import them into a **PostgreSQL** database. This script can handle large files (> 500MB) and displays a progress bar. 

<u>Usage</u>: Save the script in a `import.sh` file and run: 

    sh import.sh

***<u>Note</u>**: you will need to create the tables beforehand and match their names with their import file.* 

<pre>
<code>
#!/bin/bash
# Import script
# This script will scan throught CSV files into the 'files' folder
# in the same directory as this script
# Create a new folder to store our clean CSV files
mkdir -p files_clean

# Get current path
BASE_PATH=`dirname "$(readlink -f "$0")"`

# Database declaration
DB_NAME=your_database_name
export PGPASSWORD='your_password'

# Directories for files
FILES=files/*.csv
FILES_CLEAN=files_clean/*.csv
COUNT_FILES=`find ./files -type f -name '*.csv' | wc -l`
COUNT_FILES_CLEAN=`find ./files_clean -type f -name '*.csv' | wc -l`
i=1

for f in $FILES
do
  echo "Processing $f file..."
  echo -n "["
  for ((j=0; j<i; j++)) ; do echo -n '#'; done
  for ((j=i; j<$COUNT_FILES; j++)) ; do echo -n ' '; done
  echo -n "] $i / $COUNT_FILES" $'\r'
  ((i++))
  sleep 1

  # Cleanup CSV input
  # Explanation:
  # -e "s/\"\"/\(null\)/g": what we do here is replace empty values with (null) for COPY to handle better
  # -e "s/\"([0-9]*\S)\"/\1/g": removes quotes for numeric values
  # -e '1d' removes the first line containing headers
  # You can add as many expressions as you want by using additional -e
  # See: http://www.gnu.org/software/sed/manual/sed.html#sed-Programs for more info  
  sed -r -e '1d' -e "s/\"\"/\(null\)/g" -e "s/\"([0-9]*\S)\"/\1/g" $f > "./files_clean/$(basename "$f" .csv).csv"
done

i=1

for f in $FILES_CLEAN
do
  echo "Importing $f file into database..."
  echo -n "["
  for ((j=0; j<i; j++)) ; do echo -n '#'; done
  for ((j=i; j<$COUNT_FILES_CLEAN; j++)) ; do echo -n ' '; done
  echo -n "] $i / $COUNT_FILES_CLEAN" $'\r'
  ((i++))
  sleep 1

  # Import into database
  # COPY syntax tested on PostgreSQL 9.4, should work on > 9.X
  psql -U postgres -d $DB_NAME -c "COPY $(basename "$f" .csv) FROM '$BASE_PATH/$f' WITH (FORMAT CSV, DELIMITER ',', NULL ' (null)');"
done

exit 0
</code>
</pre>
