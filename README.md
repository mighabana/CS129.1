# CS 129.1 Final Project

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

  `$ mongod --replSet replicate --dbpath mongo1 --port 27017 --rest

  $ mongod --replSet replicate --dbpath mongo2 --port 27018 --rest

  $ mongod --replSet replicate --dbpath mongo3 --port 27019 --rest`

3. Login to a mongo instance
  `$ mongo localhost:27017`

4. Create a config file within the mongo instance

- Create the config variable

` var config = {
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
`

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
