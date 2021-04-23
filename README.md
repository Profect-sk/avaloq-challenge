# Avaloq-challenge
[![Build Status](http://www.v2202102139782142273.nicesrv.de:8080/buildStatus/icon?job=avaloq-challenge%2Fdevelop)](http://www.v2202102139782142273.nicesrv.de:8080/job/avaloq-challenge/job/develop/)
[![Quality Gate Status](http://v2202102139782142273.nicesrv.de:9000/api/project_badges/measure?project=avaloq-challenge-develop&metric=alert_status)](http://v2202102139782142273.nicesrv.de:9000/dashboard?id=avaloq-challenge-develop)
[![Coverage](http://v2202102139782142273.nicesrv.de:9000/api/project_badges/measure?project=avaloq-challenge-develop&metric=coverage)](http://v2202102139782142273.nicesrv.de:9000/dashboard?id=avaloq-challenge-develop)
[![Code Smells](http://v2202102139782142273.nicesrv.de:9000/api/project_badges/measure?project=avaloq-challenge-develop&metric=code_smells)](http://v2202102139782142273.nicesrv.de:9000/dashboard?id=avaloq-challenge-develop)
[![Bugs](http://v2202102139782142273.nicesrv.de:9000/api/project_badges/measure?project=avaloq-challenge-develop&metric=bugs)](http://v2202102139782142273.nicesrv.de:9000/dashboard?id=avaloq-challenge-develop)
[![Vulnerabilities](http://v2202102139782142273.nicesrv.de:9000/api/project_badges/measure?project=avaloq-challenge-develop&metric=vulnerabilities)](http://v2202102139782142273.nicesrv.de:9000/dashboard?id=avaloq-challenge-develop)
The `model` package contains the `Dice` class which is then extended by the `DiceDistribution`
and `DiceSimulation` classes. `DiceDistribution` is being stored in the database. 
The distribution of the roll sums and roll sum counts is being stored as a Collection Table.
The `DiceSimulation` is being used as the response of the `/dice-simulation-list` endpoint.  
  
H2 in memory db is being used. 
  
Unit and integration tests are present in the `test` folder.  
  
The solution consists of three endpoints:  
`GET /api/dice-distribution` 
with query parameters: `diceNumber, sideNumber, rollNumber`  
  
returns a JSON where the key is the sum of the roll and the value is the how many times this sum has occured.  Example for 2 dice with 6 sides and 200 rolls:  
```json
{
"2": 6,
"3": 8,
"4": 20,
"5": 21,
"6": 23,
"7": 32,
"8": 29,
"9": 20,
"10": 24,
"11": 10,
"12": 7
}
```
I was thinking about returning it as a list of objects. These objects would have two properties rollSum and count.
Then I thought about it and decided to go with the key value solution because working with a 
HasMap is easier than a list of objects especially when searching if a key already exits and so on.  

`GET /api/dice-simulation-list`  
  
Returns a JSON array of Dice Simulation Objects. Example:
```json
[
    {
        "diceNumber": 2,
        "sideNumber": 6,
        "totalNumberOfRolls": 400,
        "simulationNumber": 2
    },
    {
        "diceNumber": 3,
        "sideNumber": 6,
        "totalNumberOfRolls": 200,
        "simulationNumber": 1
    }
]
```

Firstly a distinct list of dice number and sides number is fetched from the database. 
Then this list is iterated through and using db queries the total number of rolls and simulation number is fetched.
I was thinking to query the objects with the appropriate dice number and side number and then compute 
the total number of rolls and simulation number from these objects. 
In the end the direct query for these values seemed like a more elegant solution.

`GET /api/relative-dice-distribution`
with query parameters: `diceNumber, sideNumber`
  
Returns a JSON with key and value similar to the one mentioned earlier, but the value is now a percentage.
Example:
```json
{
    "2": 3.00,
    "3": 5.75,
    "4": 7.50,
    "5": 9.50,
    "6": 12.00,
    "7": 17.75,
    "8": 14.00,
    "9": 12.25,
    "10": 8.25,
    "11": 7.50,
    "12": 2.50
}
```
Firstly a list of Dice distribution objects is fetched from the database also the total sum of rolls is fetched from the db.
Then a HashMap is created which is the sum of all the fetched dice distributions.
After that the percentage is created, and it is put in a HashMap.
