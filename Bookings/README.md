# Bookings

**Bookings** is a web application written in [ReactJS](https://reactjs.org) and [NodeJS](https://nodejs.org) that, backed by the power of the [MariaDB Node Connector](https://github.com/MariaDB/mariadb-connector-nodejs) and the [MariaDB X4 Platform](https://mariadb.com/resources/blog/deploying-mariadb-platform-x4/#smart), unleashes the power of [smart transactions](https://mariadb.com/resources/blog/introducing-mariadb-platform-x4-smart-transactions-and-cloud-native-storage/) on hundreds of millions of records with sub-second query performance without having to add any indexes!

<p align="center" spacing="10">
    <img src="media/demo.gif" style="border: 1px solid #000" />
</p>

The following will walk you through the steps for getting this application up and running (locally) within minutes! This application is completely open source. Please feel free to use it and the source code as you see fit! 

# Table of Contents
1. [Environment and Compatibility](#compatibility)
2. [Getting started with MariaDB and Hybrid Transactional-Analytical Processing](#overview)
    1. [The Basics](#intro-mariadb)
    2. [Deploying MariaDB HTAP](#installation)
    3. [Create the schema](#create-schema)
    4. [Load Flight data](#load-data)
    5. [Setting up replication](#replication)
3. [Requirements to run the app](#requirements)
4. [Getting started with the app](#getting-started)
    1. [Grab the code](#grab-code)
    2. [Build the code](#build-code)
    3. [Run the app](#run-app)
5. [Smart Transactions](#smart-transactions)
6. [Cross-Engine Queries](#cross-engine-queries)
7. [Support and Contribution](#support-contribution)
8. [License](#license)

## Environment and Compatibility <a name="compatibility"></a>

This sample was created using the following techologies:

* [MariaDB X4 HTAP (v10.4)](https://github.com/mariadb-corporation/mariadb-columnstore-htap)
* [React.js (v.16.12.0)](https://github.com/facebook/react/blob/master/CHANGELOG.md#16120-november-14-2019)
* [Node.js (v.12.x)](https://nodejs.org/docs/latest-v12.x/api/index.html)
* [NPM (v.6.11.3)](https://docs.npmjs.com/)

This sample was tested on macOS v.10.14.6.

## Overview <a name="overview"></a>

### Introduction to MariaDB <a name="intro-mariadb"></a>

[MariaDB platform](https://mariadb.com/products/mariadb-platform/) unifies [MariaDB TX (transactions)](https://mariadb.com/products/mariadb-platform-transactional/) and [MariaDB AX (analytics)](https://mariadb.com/products/mariadb-platform-analytical/) so transactional applications can retain unlimited historical data and leverage powerful, real-time analytics in order to provide data-driven customers with more information, actionable insight and greater value – and businesses with endless ways to monetize data. It is the enterprise open source database for hybrid transactional/analytical processing at scale.

<p align="center">
    <img src="media/platform.png" />
</p>

### Deploying MariaDB Hybrid Transactional-Analytical Processing (HTAP) <a name="installation"></a>

MariaDB Platform supports [Hybrid Transactional-Analytical Processing (HTAP)](https://mariadb.com/docs/deploy/htap/) through a combination of MariaDB Enterprise Server, MariaDB ColumnStore, and MariaDB MaxScale.

Here's a simple architecture diagram of MariaDB X4 Platform.

<p align="center" spacing="10">
    <img src="media/x4.png" style="height:400px;" />
</p>

For this application we'll be targeting a single instance of MariaDB HTAP, and instructions for setting such an environment can be found here:

https://github.com/mariadb-corporation/mariadb-columnstore-htap

**Note:** You can also find more details on how to deploy MariaDB X4 [here](https://mariadb.com/docs/deploy/htap/).

### Create the schema <a name="create-schema"></a>

Execute [create.sql](schema/create.sql) within your instance of MariaDB X4.

### Loading Flights data into ColumnStore <a name="load-data"></a>

Instructions on retrieving and importing the flights dataset into a MariaDB ColumnStore database (within HTAP) can be [here](https://github.com/mariadb-corporation/mariadb-columnstore-samples/tree/master/flights). Please note that he scripts provided within that repository only targets data for the year 2019 (~7.5 million records). 

**Note** If you'd like to retrieve data spanning from 1990 to 2020 (~182 million records) please use the following scripts in place of the ones provided [here](https://github.com/mariadb-corporation/mariadb-columnstore-samples/tree/master/flights).

* [get_flight_data.sh](data/get_flight_data.sh)
* [load_flights_data.sh](data/load_flights_data.sh)

Of course, you can also modify the scripts to fit your needs.

### Setting up HTAP Replication <a name="htap-replication"></a>

Using MariaDB Replication, MariaDB Enterprise Server replicates writes from InnoDB tables to the MariaDB ColumnStore tables, ensuring that the application can perform analytical processing on current data. Combining MariaDB Replication with MariaDB MaxScale configured as a Binlog Server, MariaDB Enterprise Server can host InnoDB and ColumnStore on the same Server.

This application uses replication on a single table called "Flights", which exists in the InnoDB and ColumnStore instances. In order to set up replication add the following to `/etc/maxscale.cnf` for your HTAP instance.

```
[replication-filter]
type         = filter
module       = binlogfilter
match        = /[.]flights/
rewrite_src  = innodb
rewrite_dest = columnstore
```

For more information on configuring MariaDB HTAP please review the official [Enterprise Documentation](https://mariadb.com/docs/deploy/htap/#maxscale-configuration).

## Requirements to run the app <a name="requirements"></a>

This project assumes you have familiarity with building web applications using [ReactJS](https://reactjs.org) and [NodeJS](https://nodejs.org) technologies. 

The following is required to run this application:

1. [Download and install MariaDB HTAP](https://mariadb.com/docs/deploy/htap/). 
2. [Download and install NodeJS](https://nodejs.org/en/download/).
3. git (Optional) - this is required if you would prefer to pull the source code from GitHub repo.
    - Create a [free github account](https://github.com/) if you don’t already have one
    - git can be downloaded from git-scm.org

## Getting started with the app <a name="getting-started"></a>

### Grab the code <a name="grab-code"></a>

Download this code directly or use [git](git-scm.org) (through CLI or a client) to retrieve the code.

### Configure the code <a name="configure-code"></a>

Configure the MariaDB connection by [adding an .env file to the Node.js project](https://github.com/mariadb-corporation/mariadb-connector-nodejs/blob/master/documentation/promise-api.md#security-consideration).

Example implementation:

```
DB_HOST=<host_address>
DB_PORT=<port_number>
DB_USER=<username>
DB_PASS=<password>
DB_NAME=<database>
```

### Build the code <a name="build-code"></a>

Once you have retrieved a copy of the code you're ready to build and run the project! However, before running the code it's important to point out that the application uses several Node Packages.

Executing the CLI command `npm install` within the [src](src) AND [client](src/client) folders will target the the relative `package.json` file and install all dependencies.


For more information on `npm install` check out the [Official Node.js Documentation](https://docs.npmjs.com/downloading-and-installing-packages-locally).

### Run the app <a name="run-app"></a>

Once you've pulled down the code and have verified that all of the required Node packages are installed you're ready to run the application! It's as easy as 1,2,3.

1. Using a command line interface (CLI) navigate to the `src` directory.

<p align="center">
    <img src="media/cli_root.png" style="border: 1px solid #000" />
</p>

2. Run the command:

```bash
npm start
```

<p align="center">
    <img src="media/npm_start.png" style="border: 1px solid #000" />
</p>

3. Open a browser window and navigate to http://localhost:3000.

<p align="center">
    <img src="media/get_started.png" style="border: 1px solid #000" />
</p>

## Smart Transactions <a name="smart-transactions"></a>

At this point you're probably wondering, what are smart transactions?

At their core, smart transactions are the standard transactions that databases have been performing for decades – ultimately powering the online interactions we’ve become accustomed to. The difference with modern applications is the use of real-time analytics before, during and/or after these transactions.

**Pre-transaction**

This application uses real-time analytics before a flight is booked. Each flight ticket option contains information calculated from the historical records (average delay, average duration, flight score, etc.) within the `flights` table.

<p align="center">
    <img src="media/flight_1.png" style="border: 1px solid #000" />
</p>

<p align="center">
    <img src="media/flight_2.png" style="border: 1px solid #000" />
</p>

**Post-transaction**

This application also uses real-time analytics after a flight has been booked, and a trip has been created. 

<p align="center">
    <img src="media/trip_1.png" style="border: 1px solid #000" />
</p>

## Cross-Engine Queries <a name="cross-engine-queries"></a>

This application uses cross-engine queries to maximize the potentials of the MariaDB X4 Platform. Cross-engine querying is the ability to access, via MaxScale, both the transactional and analytics data within a single query.  

<p align="center">
    <img src="media/cross_engine.png" style="border: 1px solid #000" />
</p>

## Support and Contribution <a name="support-contribution"></a>

Thanks so much for taking a look at the Bookings app! As this is a very simple example, there's plenty of potential for customization. Please feel free to submit PR's to the project to include your modifications!

If you have any questions, comments, or would like to contribute to this or future projects like this please reach out to us directly at developers@mariadb.com or on [Twitter](https://twitter.com/mariadb).

## License <a name="license"></a>
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg?style=plastic)](https://opensource.org/licenses/Apache-2.0)