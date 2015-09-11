mobWell.login
==

This project creates an easy and reusable secure platform to make login into mobwell servers.

## Files

* config.inc  
* database.sql  
* login.php  
* logout.php  
* register.php  
* req_reg.php  
* util.inc

### config.inc

Place your database configurations here;

### database.sql

The script that contains the skeleton of your database and the necessary triggers;
  

### req_reg.php
Claims for the salt of a new record.

| I/O | Meaning | |
|------|---------|-----|
| **input** |   | |
|           |   | |
| **output**|   | |
| JSON      |   | |
|           | salt | The salt that identifies the new user|


### register.php
Registers the user in the system.

I/O | Meaning | |
|------|---------|--------|
| **input** |   |
| salt      | The salt of the requested user   |
| login     | the new login|
| password  | The encrypted password |
| ofields   | Other user fielkds, like photo, address, etc.|
| **output**|   ||
| JSON      |   ||
|           | salt | The salt that identifies the new user|
|           | errcode   | * 400. Missing parameter|
|           |           | * 401. Login exists already|
|           |           | * 200. OK|


### login.php
The login script. Automatically register last login date, and returns an active token

| I/O       | Meaning ||
------|---------|------|
| **input** | |
| login     | the users's login||
| phs       | The phase of the login: |
|           | 1. To get the salt |
|           | 2. To enter the password with the salt.||
| password  | _Only required if in phase 2._ |
|           | Password is the MD5 of the user's password and the hash reversed||
| **output**|   ||
| JSON      |   ||
|           | active_token | The token that identifies the client to use between communications |
|           | errorCode | 404 - bad Login; |
|           |           | 401 - Wrong Password;| 
|           |           | 200 - OK) |


###logout.php
Performs a simple logout, and cleans the token.

| I/O       | Meaning ||
|-----------|---------|----------|
| **input** |   |
| token     | the token to be cleaned   |
| **output**|   ||
| JSON      |   ||
|           | errcode   | 400. Missing a parameter |
|           |           | 204. Token not found.|
|           |           | 200. OK|

## How it works

### Register

1. Request a salt for the new user 
    * _req_request.php_
2. With the new salt, register the user (the salt is unique) 
    * _register.php_?salt=HASH&login=picordino&password=MD5PWD&name=picordino&nick=carrapito&...

### Login
1. Login in phase one, to request the salt of the introduced Login parameter.
    * -> login.php?phs=1&login=picordino
    * <- HASH SALT (*only useful to generate the hash of the user password*)
2. In client side, and **with the salt you just received**, make an MD5 of your password and the *reversed salt*
    * *MD5PWD=MD5(user_password+stringReverse(salt);*
3. Login in Phase two, to receive a token that makes proof of who you are, and handles the communication
    * ->*login.php?phs=2&login=picordino&password=MD5PWD*
    * <- ACTIVETOKEN
4. Logout
    * logout.php?token=TOKEN
