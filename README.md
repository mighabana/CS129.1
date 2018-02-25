# CS 129.1 Final Project

Repository for the final project for CS 129.1 Contemporary Databases. For our final project we were tasked to handle a large dataset, and organize the data through a series of MapReduce functions. Additionally we were tasked with replicating and sharding the data into multiple mongo databases.

## Loading the Dataset

- The first dataset that we will be using can be downloaded from Kaggle (https://www.kaggle.com/dgawlik/nyse)
- From the downloaded dataset we will be using the prices.csv for the data regarding the stock prices and their respective volumes

- The next dataset to segregate the stocks by their subsectors the data will be downloaded from the NASDAQ website (http://www.nasdaq.com/screening/companies-by-industry.aspx?exchange=NYSE)
- Lastly place these two files within your project folder

## Setting up the Replicate sets

1. Set-up the project folder

- Create a folder named replicate

  `$ mkdir replicate`

- Change the current directory to replicate

  `$ cd replicate`

- Create three folders within replicate named mongo1, mongo2, and mongo3

  `$ mkdir mongo1`

  `$ mkdir mongo2`

  `$ mkdir mongo3`

2. Running the servers

- Run the following commands in separate terminal windows to initialize each servers

  `$ mongod --replSet replicate --dbpath mongo1 --port 27017 --rest`

  `$ mongod --replSet replicate --dbpath mongo2 --port 27018 --rest`

  `$ mongod --replSet replicate --dbpath mongo3 --port 27019 --rest`

3. Login to a mongo instance

  `$ mongo localhost:27017`

4. Create a config file within the mongo instance

- Create the config variable

```javascript
var config = {
  "_id": "replicate",
  "version" : 1,
  "members" :   [
    {
      "_id" : 0,
      "host" : "localhost:27017",
      "priority" : 1
    },
    {
      "_id" : 1,
      "host" : "localhost:27018",
      "priority" : 0
    },
    {
      "_id" : 2,
      "host" : "localhost:27019",
      "priority" : 0
    }
  ]
}
```

- Initialize the replica set using the config variable

`rs.intiate(config)`

5. Loading the dataset into the mongo instance

- Run a new terminal in the folder of the datasets that you want to be uploaded
- Then run the following command

  `$ mongoimport --db <dbname> --collection <collection_name> --type csv --file <filename.csv> -h localhost:27017`

6. Check if the files have been loaded into the mongo instance

- In the mongo instance terminal

` > show dbs`

` > use <dbname>`

` >db.<collection_name>.find().pretty()`

- If data appears after running the last command then the dataset has been successfully loaded into the mongo instance

7. Check if the data has successfully been replicated

- Login to another mongo instance

`$ mongo localhost:27018`

- Once logged-in set the current terminal as a slave

`> rs.slaveOk()`

- Then use the loaded database

`> use <dbname>`

- Check if the data is consistent between both mongo instances

`> db.<collection_name>.find().pretty()`

## Executing the MapReduce Functions

1. MapReduce function for the prices

```
mapP = function() {
  emit({
    symbol: this.symbol
  },{
    count: 1,
    stock: 'Date: ' + this.date + '; Close: ' + this.close
  });
}

reduceP = function(key, values) {
  var total = 0;
  var stock = [];
  for(var i = 0; i < values.length; i++) {
    total += values[i].count;
    stock[i] = values[i].stock;
  }
  return {count: total, stock: stock};
}
```

2. MapReduce function for the companylist

```
mapCompanies = function() {
  emit({
    subsectors: this.Sector
  },{
    count: 1,
    symbol: this.Symbol
  });
}

reduceCompanies = function(key,values) {
  var total = 0;
  var symbols = [];
  for(i = 0; i < values.length; i++) {
    symbols[i] = values[i].symbol;
    total += values[i].count;
  }
  return {count: total, symbol: symbols};
}
```

## Running the MapReduce Functions

1. To run the mapReduce function for the prices:

```
resultsPrice = db.runCommand({
  mapReduce: 'prices',
  map: mapP,
  reduce: reduceP,
  out: 'prices.reduced'
});

db.prices.reduced.find().pretty()
```

2. To run the mapReduce function for the companylist:

```
resultsCompanies = db.runCommand({
  mapReduce:'companylist',
  map: mapCompanies,
  reduce: reduceCompanies,
  out: 'companylist.reduced'
});

db.companylist.reduced.find().pretty()
```

## Sharding the Dataset

1. Enter the project folder

2. Once in the project folder create the necessary directories for sharding

- These directories are the config, the nodes, and the mongos directory

`mkdir config`

`mkdir node1`

`mkdir node2`

`mkdir mongos`

3. After creating the necessary directories we need to setup the servers for each

- To set up the config server:

`mongod --configsvr --replSet finals --dbpath config --port 27017`

- To set up the shards:

`mongod --shardsvr --dbpath node1 --port 27018`

`mongod --shardsvr --dbpath node2 --port 27019`

4. Then setup the sharding configuration within the config server

- First connect to the config server

`mongo localhost:27017`

- then initiate the replicate set configuration

`rs.initiate()`

5. Then set up the mongos server and add the shards

- set up the mongos server:

`mongos --configdb finals/localhost:27017 --port 27020`

- then connect to the mongos server:

`mongo localhost:27020`

- reduce the default chunkSize:

`use config`

`db.settings.save( { _id:"chunksize", value: 1 } )`

- add the shards:

`sh.addShard("localhost:27018")`

`sh.addShard("localhost:27019")`

6. Lastly import the data into the mongos server

`mongoimport --db finals --collection prices --type csv --headerline --file prices.csv --host localhost:27020`
