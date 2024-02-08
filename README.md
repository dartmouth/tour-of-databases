
# A tour of databases - Research data storage options

Author: Elijah Gagne

Last updated: February 2024

## Overview

- Why a database
- Types of databases
  1. Hierarchical Databases
  2. Object oriented databases
  3. Relational Databases
  4. Non-Relational Databases
- Examples
  - Download example dataset
  - Example: Postgres
  - Example: MariaDB
  - Example: MongoDB
  - Example: Redis
  - Clean up examples
- How to run them for HPC
- Questions and comments

Credit for types: https://www.mongodb.com/databases/types

## Why a database

1. Concurrency
2. Efficiency
3. Other
  - Data integrity
  - Separation of concerns
  - Security
  - Organization
  - Built in tools

## Hierarchical Databases

Design: a tree structure where each node has one parent and possibly multiple children.

Examples:
- File system
- Windows Registry
- IBM Information Management System (IMS)

## Object oriented databases

Design: OODs store data in objects similar to those used in object-oriented programming, allowing complex data structures to be represented directly in the database

Examples
- Objectivity/DB
- IBM Db2
- GemStone/S

## Relational Databases

Design: data is stored in a tables and linked by defining relationships

Examples
- Postgres
- MariaDB
- SQL Server

## Non-Relational Databases (aka NoSQL)

Design: data is stored as key-value pairs, documents, graphs, or wide-column formats.

Examples
- MongoDB
- Redis
- Elasticsearch

## Download a dataset

- Browse to https://www.kaggle.com/datasets/kanchana1990/2024-amazon-best-sellers-top-valentine-gifts
- Click "Download"

```sh
# Unzip a test dataset
cd ~/Downloads
unzip archive.zip
rm -f archive.zip
```

## Example: Postgres

```sh
# Install a GUI client for Postgres, MariaDB, and other common DBs
brew install --cask dbeaver-community

# Install the command line client
brew install libpq
# Put them in the PATH (breaks ability to install DB engine)
brew link --force libpq

# Run a container for Postgres
docker run -d \
--name some-postgres \
-p 5432:5432 \
-e POSTGRES_PASSWORD=mysecretpassword \
postgres

# Copy our dataset into the container
docker cp $HOME/Downloads/amazon_2024_valentines_best_sellers.csv some-postgres:/tmp

# Connect command line
export PGPASSWORD=mysecretpassword
psql \
-h localhost \
-p 5432 \
-d "dbname=postgres" \
-U "postgres" \
--set=keepalives_idle=10
```

```sql
-- Create a database and swich to it
create database example;
\connect example

-- Create a table
CREATE TABLE top_valentine_gifts (
  title VARCHAR(512) UNIQUE NOT NULL,
  brand VARCHAR(64),
  description VARCHAR(2048),
  starsBreakdown_3star NUMERIC(5,3) NOT NULL,
  starsBreakdown_4star NUMERIC(5,3) NOT NULL,
  starsBreakdown_5star NUMERIC(5,3) NOT NULL,
  reviewsCount INTEGER,
  price NUMERIC(10,2),
  price_currency VARCHAR(4),
  price_value NUMERIC(10,2),
  categoryPageData_productPosition INTEGER NOT NULL
);

-- Import the records
COPY top_valentine_gifts(title, brand, description, starsBreakdown_3star, starsBreakdown_4star, starsBreakdown_5star, reviewsCount, price, price_currency, price_value, categoryPageData_productPosition)
FROM '/tmp/amazon_2024_valentines_best_sellers.csv'
DELIMITER ','
CSV HEADER;

-- Query the records
select * from top_valentine_gifts;
```

Look at the data in DBeaver

Reference
- https://hub.docker.com/_/postgres

## Example: MariaDB

```sh
# Install the command line client
# Use MySQL on a Mac since that client is simpler to install
brew install mysql-client

# Run a container for MariaDB
docker run -d \
--name some-mariadb \
-p 3306:3306 \
-e MARIADB_ROOT_PASSWORD=my-secret-pw \
mariadb

# Copy our dataset into the container
docker cp $HOME/Downloads/amazon_2024_valentines_best_sellers.csv some-mariadb:/tmp

# Connect command line
mysql \
--user=root \
--password=my-secret-pw \
--host=127.0.0.1 \
--port=3306
```

```sql
-- Create a database and swich to it
create database example;
use example

-- Create a table
CREATE TABLE top_valentine_gifts (
  title VARCHAR(1024) UNIQUE NOT NULL,
  brand VARCHAR(1024),
  description VARCHAR(4096),
  starsBreakdown_3star DECIMAL(5,3) NOT NULL,
  starsBreakdown_4star DECIMAL(5,3) NOT NULL,
  starsBreakdown_5star DECIMAL(5,3) NOT NULL,
  reviewsCount INTEGER,
  price DECIMAL(10,2),
  price_currency VARCHAR(4),
  price_value DECIMAL(10,2),
  categoryPageData_productPosition INTEGER NOT NULL
);

-- Import the records
LOAD DATA INFILE '/tmp/amazon_2024_valentines_best_sellers.csv'
INTO TABLE top_valentine_gifts
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 LINES
(title, @vbrand, @vdescription, starsBreakdown_3star, starsBreakdown_4star, starsBreakdown_5star, @vreviewsCount, @vprice, @vprice_currency, @vprice_value, categoryPageData_productPosition)
SET
brand = NULLIF(@vbrand,''),
description = NULLIF(@vdescription,''),
price = NULLIF(@vprice,''),
price_currency = NULLIF(@vprice_currency,''),
price_value = NULLIF(@vprice_value,''),
reviewsCount = NULLIF(@vreviewsCount,'');

-- Query the records
select * from top_valentine_gifts;
```

Look at the data in DBeaver

Reference
- https://hub.docker.com/_/mariadb

## Example: MongoDB

```sh
# Install a GUI client for MongoDB
brew install --cask mongodb-compass

# Install MongoDB and the client
brew tap mongodb/brew
brew install mongodb-community

# Run a container for MongoDB
docker run -d \
--name some-mongo \
-e MONGO_INITDB_ROOT_USERNAME=mongoadmin \
-e MONGO_INITDB_ROOT_PASSWORD=secret \
-p 27017:27017 \
mongo

# Connect command line
mongosh "mongodb://localhost:27017" \
--username mongoadmin \
--authenticationDatabase admin
```

```json
// Create and switch to a new database
use blog

// Create a collection
db.createCollection("posts")

// Insert a record
db.posts.insertOne({
  title: "Post Title 1",
  body: "Body of post.",
  category: "News",
  likes: 1,
  tags: ["news", "events"],
  date: Date()
})

// Insert a few more
db.posts.insertMany([
  {
    title: "Post Title 2",
    body: "Body of post.",
    category: "Event",
    likes: 2,
    tags: ["news", "events"],
    date: Date()
  },
  {
    title: "Post Title 3",
    body: "Body of post.",
    category: "Technology",
    likes: 3,
    tags: ["news", "events"],
    date: Date()
  },
  {
    title: "Post Title 4",
    body: "Body of post.",
    category: "Event",
    likes: 4,
    tags: ["news", "events"],
    date: Date()
  }
])

// List out the records
db.posts.find()
```

Look at the data in MongoDB Compass

```json
{title: 'Post Title 1'}
```

References
- https://hub.docker.com/_/mongo
- https://www.w3schools.com/mongodb/mongodb_mongosh_create_database.php

## Example: Redis

```sh
# Install Redis
brew install redis

# Run a container for Redis
docker run -d \
--name some-redis \
-p 6379:6379 \
redis redis-server --requirepass "SUPER_SECRET_PASSWORD"

# Connect command line
redis-cli -h 127.0.0.1 -p 6379 -a SUPER_SECRET_PASSWORD
```

```batch
ECHO "Create a key/value pair"
SET date1 2024-02-08

ECHO "Retrieve the entry"
GET date1

ECHO "Rename it"
RENAME date1 date0

ECHO "List the entries"
KEYS date0

ECHO "Delete the entry"
DEL date0

ECHO "Confirm it's gone"
KEYS date0
```

References
- https://hub.docker.com/_/redis
- https://www.tutorialspoint.com/redis/redis_commands.htm

## Clean up examples

```sh
docker rm -f some-redis
docker rm -f some-mongo
docker rm -f some-mariadb
docker rm -f some-postgres
```

## How to run them for HPC

https://dashboard.dartmouth.edu/

Email research.computing@dartmouth.edu for additional help

## Questions and comments

http://dartgo.org/feedback

https://libcal.dartmouth.edu/event/11750385

