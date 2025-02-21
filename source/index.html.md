---
title: Octo documentation

language_tabs: # must be one of https://git.io/vQNgJ
  - bash

toc_footers:
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

search: true

code_clipboard: true
---

# Introduction

Welcome to the Octo! You can use this tools to register and display all applications versions of your compagny for each client and platform.

# Installation

Octo is composed of three components:

* Database: postgresql version 13.3
* API: [octo-spy](https://github.com/Zorin95670/octo-spy/tree/1.14.0) made in Java
* Web application: [octo-board](https://github.com/Zorin95670/octo-board/tree/2.10.0) made in Vue-js

## Docker
> Build `octo-spy` and `octo-board` docker images:

```bash
docker build -t octo-spy octo-spy/

docker build -t octo-board octo-board/
```

To use Octo with docker you must build `octo-spy` and `octo-board` images.

Once images build, you can setup your environment like the [compose example](https://github.com/Zorin95670/octo/blob/1.6.0/docker-compose.yml).

Don't forget to setup volume for database.

## Other

If you didn't have a proper docker environment. You can build `octo-spy` and `octo-board` and deploy them on associated service.

### Build octo-spy

> Command to build `octo-spy`, run it in `./octo-spy` folder

```bash
mvn clean package -Dmaven.test.skip=true
```

To build `octo-spy` you need `java 16` and `maven 3.8.1` on your building machine.

You can find the associated war in `target` folder.
And you can deploy it on [Tomcat (v9.0.52)](http://tomcat.apache.org/).

<aside class="notice">
Before running API for the first time, you must initialize your database with this script
<code>src/main/resources/db/init.sql</code>.
</aside>

### Build octo-board

> Commands to build `octo-board`, run it in `./octo-board` folder

```bash
# Install project dependencies
npm install --legacy-peer-deps

# Build changelog page and json
npm run changelog --silent > public/changelog.html
RUN npm run changelogToJson

# Build project
npm run build
```

To build `octo-board` you need `node 14.17.X` and `npm 7.20.X` on your building machine.

You can find the associated war in `build` folder.
And you can deploy it on your wanted HTTP server.

<aside class="warning">
You must configure your HTTP server to make redirection on your server api.
You can find an exemple for nginx <a href="https://github.com/Zorin95670/octo/blob/1.1.0/nginx.conf">here</a>.
</aside>

# Configuration

## Default user

Default administrator login is `admin` with `admin` as password and has `no-reply@change.it` as default e-mail.

<aside class="warning">
You must change default password and e-mail in application settings on <code>octo-board</code>.
</aside>

## Database connection

Default user login is `octo`.
Default user password is `password`.

You can override application settings with environment variable:

Variable name | default value | Description
------------- | ------------- | -----------
octo-spy.database.host | localhost | Database address.
octo-spy.database.port | 5432 | Database port.
octo-spy.database.name | octo_db | Database name.
octo-spy.database.username | octo | Database user login.
octo-spy.database.password | password | Database user password.

# Usages

## Authentication

### Basic authentication

> Authentication example to get user information

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --request GET                                \
  http://spy:8080/octo-spy/api/users/me
```

You need to encode in base64 your `user:password`.

### Token authentication

> Authentication example to get user information

```bash
curl                                           \
  --header "Authorization: Token TOKEN" \
  --request GET                                \
  http://spy:8080/octo-spy/api/users/me
```

You need to use token given by application when you create a token.

## Create master and sub-project

Go to octo-board, as administrator go to master project page, to create master project.

To create sub-project, still as administrator, go to master project page and go to sub-project page.


## Register `in progress` deployment

> Register new `in progress` deployment for octo-spy version 1.0.0 in production for SomeClient

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --header "Content-Type: application/json"    \
  --request POST                               \
  --data 'DATA_TO_SEND'                        \
  http://spy:8080/octo-spy/api/deployments
```

> Data to send:

```json
{
  "environment": "Production",
  "project": "octo-spy",
  "version": "1.0.0",
  "client": "SomeClient",
  "alive" : true,
  "inProgress": true
}
```

A deployment is a record of new version on a project for a specific environment.

Default environments:

* Development
* Integration
* Pre-production
* Production

You can directly indicate if your deployment is in progress or not.

## Stop progress of a deployment

> Delete progress state for octo-spy version 1.0.0 in production for SomeClient

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --header "Content-Type: application/json"    \
  --request DELETE                             \
  --data 'DATA_TO_SEND'                        \
  http://spy:8080/octo-spy/api/deployments/progress
```

> Data to send:

```json
{
  "environment": "Production",
  "project": "octo-spy",
  "version": "1.0.0",
  "client": "SomeClient",
  "alive" : true
}
```

At the end of deployment you can indicate, that is done by deleting the progress state.

# API documentation

## Default object

Default resource object:

```json
{
  "total": 0,
  "page": 0,
  "count": 0,
  "resources": []
}
```

Key | Type | Description
--- | ---- | -----------
total | Number | Total of resources in database
page | Number | Current index of pagination
count | Number | Length of resources, maximum 200
resources | Array | Array of wanted resource

Default error object:

```json
{
  "message": null,
  "field": null,
  "value": null,
  "cause": null
}
```

Key | Type | Description
--- | ---- | -----------
message | String | Generic error's message.
field | String | Field name where error occurs
fields | String | Field name where error occurs
value | String | Field value or error explanation
cause | String | Stack trace of the error.

<aside class="notice">Fields can only be used with the report endpoint.</aside>
<aside class="notice">
  Fields corresponding to multiple field, you can declare multiple fields in query parameters with it.
  <br/>
  Example:
  <br/>
  <code>/report/deployments?fields=project&fields=client</code>
</aside>

## Query filter explanation

On some endpoints, you can use search field to filter your resources.

You have 4 types of search: `TEXT`, `DATE`, `NUMBER` and `BOOLEAN`.

### Default modifier

There is a modifier that can be applied on all searches types:

* `not_`: 'Not' operator for the search, which applies negation:
  * `fieldName=value`: Return all data that match `value`
  * `fieldName=not_value`: Return all data that different than `value`
* `|`: 'Or' operator that makes a list:
  * `fieldName=value1|value2`: Return all data that match `value1` or `value2`
  * `fieldName=not_value1|value2`: Return all data that different than `value1` or equals to `value2`
* `null` and `not_null`

### Text search

* Equals: `fieldName=value`
* Like: `fieldName=lk_value`
  * `value` can contain `*` to symbolise any characters. It can be escape: `\*`
  * Not like: `fieldName=not_lk_value`
* Multiple equals: `fieldName=value1|value2|value3`
* Multiple like: `fieldName=lk_value1|lk_value2|lk_value3`
* All combined: `fieldName=lk_value1|not_lk_value2|value3|not_value4`

### Date search

Input date format: `yyyy-MM-dd HH:mm:ss`

Ouput date format: `yyyy/MM/dd HH:mm:ss`

* Equals: `fieldName=2000-01-01 00:00:00`
* Inferior: `fieldName=lt2000-01-01 00:00:00`
* Superior: `fieldName=gt2000-01-01 00:00:00`
* Between: `fieldName=2000-01-01 00:00:00bt2001-01-01 00:00:00`
* All combined: `fieldName=2000-01-01 00:00:00|lt2000-01-01 00:00:00`

### Number search

* Equals: `fieldName=1`
* Not equals: `fieldName=not_1`
* All combined: `fieldName=1|2|not_3`

### Boolean search

* Equals: `fieldName=true`
* Not equals: `fieldName=not_false`

## Authentication's errors

Apply for all endpoint that need authentication.

* `401 - Authentication error`: On bad authorization property
* `401 - Authentication error`: On bad authentication scheme
* `401 - Authentication has failed`: On authentication error

## Administrator

### Update administrator password

> Update administrator password

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --request PUT                                \
  --data 'BASE64_PASSWORD'                     \
  http://spy:8080/octo-spy/api/administrator/password
```

* Need to authentication: **Yes**
* Token allowed: No
* Path: `/octo-spy/api/administrator/password`
* Method: `PUT`
* Data type: `String` encoded in base64
* Success status: `204 - No content`
* Errors status:
  * `400 - Field value is empty`: On blank encoded password
  * `400 - Wrong field value`: Bad length on decoded password
  * `500 - Internal error occurred`: On no default admin account

<aside class="notice">Password length need to be between 8 and 50 characters.</aside>

### Update administrator e-mail

> Update administrator e-mail

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --request PUT                                \
  --data 'EMAIL'                               \
  http://spy:8080/octo-spy/api/administrator/email
```

* Need to authentication: **Yes**
* Token allowed: No
* Path: `/octo-spy/api/administrator/email`
* Method: `PUT`
* Data type: `String`
* Success status: `204 - No content`
* Errors status:
  * `400 - Wrong field value`: On invalid e-mail
  * `500 - Internal error occurred`: On no default admin account

## Alerts

### Get all application alerts

> Get all application alerts

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --request GET                                \
  http://spy:8080/octo-spy/api/alerts
```

> Example of success:

```json
{
  "total": 1,
  "page": 0,
  "count": 1,
  "resources": [{
    "severity": "critical",
    "type": "security",
    "message": "Administrator's password is not secure, please change it."
  }]
}
```

* Need to authentication: **Yes**
* Token allowed: No
* Path: `/octo-spy/api/alerts`
* Method: `GET`
* Success status:
  * `200 - Ok`: On alerts
  * `204 - No content`: On no alerts

Key | Type | Description
--- | ---- | -----------
severity | String | Indicates severity of alert (`critical` or `warning`)
type | String | Indicates type of alert (`security` or `incompatible`)
message | String | Explanation of alert

There is only 3 alerts:

* Having same default administrator password.
* Having same default adminstrator e-mail.
* Having a incompatible database version.

## Client

### Get all clients

> Get all application clients

```bash
curl                                  \
  --request GET                       \
  http://spy:8080/octo-spy/api/clients
```

> Example of success:

```json
["Client1", "Client2"]
```

* Need to authentication: No
* Path: `/octo-spy/api/clients`
* Method: `GET`
* Success status: `200 - Ok`

## Deployment

### Get deployment by id

> Get deployment by id

```bash
curl                                  \
  --request GET                       \
  http://spy:8080/octo-spy/api/deployments/[id]
```

> Example of success:

```json
{
  "id": 0,
  "environment": null,
  "project": null,
  "masterProject": null,
  "client": null,
  "version": null,
  "alive": true,
  "insertDate": null,
  "updateDate": null
}
```

* Need to authentication: No
* Path: `/octo-spy/api/deployments/[id]`
* Method: `GET`
* Query parameter:
  * `id`: id of deployment
* Success status: `200 - Ok`
* Errors status:
  * `400 - Field value is empty`: On blank id
  * `404 - Entity not found`: On unknown id

Key | Type | Description
--- | ---- | -----------
id | Number | Primary key
environment | String | Environment name
project | String | Project name
masterProject | String | Master project name
client | String | Client name
version | String | Deployed version
alive | Boolean | is still alive.
insertDate | Date | Creation date, format: `yyyy/MM/dd HH:mm:ss`
updateDate | Date | Last update date, format: `yyyy/MM/dd HH:mm:ss`

### Get all deployments

> Get all deployments

```bash
curl                                  \
  --request GET                       \
  http://spy:8080/octo-spy/api/deployments
```

> Example of success:

```json

{
  "total": 1,
  "page": 0,
  "count": 1,
  "resources": [{
    "id": 0,
    "environment": null,
    "project": null,
    "masterProject": null,
    "client": null,
    "version": null,
    "alive": true,
    "insertDate": null,
    "updateDate": null
  }]
}
```

* Need to authentication: No
* Path: `/octo-spy/api/deployments`
* Method: `GET`
* Success status:
  * `200 - Ok`: When all resources are returned
  * `206 - Partial content`: When partial resources are returned

Available query parameters:

Field name | Search type
---------- | -----------
id | NUMBER
environment | TEXT
project | TEXT
masterProject | TEXT
client | TEXT
version | TEXT
alive | BOOLEAN
inProgress | BOOLEAN

### Get last deployments

> Get last deployments

```bash
curl                                  \
  --request GET                       \
  http://spy:8080/octo-spy/api/deployments/last
```

> Example of success:

```json

{
  "total": 1,
  "page": 0,
  "count": 1,
  "resources": [{
    "id": 0,
    "environment": null,
    "project": null,
    "masterProject": null,
    "client": null,
    "version": null,
    "alive": true,
    "insertDate": null
  }]
}
```

* Need to authentication: No
* Path: `/octo-spy/api/deployments/last`
* Method: `GET`
* Success status:
  * `200 - Ok`: When all resources are returned
  * `206 - Partial content`: When partial resources are returned

Available query parameters:

Field name | Search type
---------- | -----------
id | NUMBER
environment | TEXT
project | TEXT
masterProject | TEXT
client | TEXT
version | TEXT
alive | BOOLEAN
inProgress | BOOLEAN
onMasterProject | BOOLEAN

### Create a deployment

> Create a deployment

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --header "Content-Type: application/json"    \
  --request POST                               \
  --data 'DATA_TO_SEND'                        \
  http://spy:8080/octo-spy/api/deployments
```

> Example of data to send:

```json
{
  "environment": "Production",
  "project": "octo-spy",
  "client": "SomeClient",
  "version": "1.0.0",
  "alive" : true,
  "inProgress" : true
}
```

> Example of success:

```json
{
  "id": 1,
  "environment": "Production",
  "project": "octo-spy",
  "masterProject": null,
  "client": "SomeClient",
  "version": "1.0.0",
  "alive": true,
  "insertDate": null,
  "updateDate": null
}
```

* Need to authentication: **Yes**
* Token allowed: **Yes**
* Path: `/octo-spy/api/deployments`
* Method: `POST`
* Data type: `JSON`
* Success status: `201 - Created`
* Errors status:
  * `400 - Field value is empty`: On blank environment, project, client or version
  * `404 - Entity not found`: On unknown environment or project

<aside class="notice">
If alive is set to <code>true</code>, this will disable previous deployment with same project, client and environment.
</aside>

Body parameters:

Key | Type | Mandatory | Description
--- | ---- | --------- | -----------
environment | String | Yes | Environment name
project | String | Yes | Project name
client | String | Yes | Client name
version | String | Yes | Deployed version
alive | Boolean | No, default `false` | Is still alive.
inProgress | Boolean | No, default `false` | Is in progress.

### Update a deployment

> Update a deployment

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --header "Content-Type: application/json"    \
  --request PATCH                              \
  --data 'DATA_TO_SEND'                        \
  http://spy:8080/octo-spy/api/deployments/[id]
```

> Example of data to send:

```json
{
  "environment": "Production",
  "project": "octo-spy",
  "client": "SomeClient",
  "version": "1.0.0",
  "alive" : true,
  "inProgress" : true
}
```

* Need to authentication: **Yes**
* Token allowed: **Yes**
* Path: `/octo-spy/api/deployments/[id]`
* Method: `PATCH`
* Query parameter:
  * `id`: id of deployment
* Data type: `JSON`
* Success status: `204 - No content`
* Errors status:
  * `400 - Field value is empty`: On blank environment, project, client or version
  * `404 - Entity not found`: On unknown environment or project

Body parameters:

Key | Type | Mandatory | Description
--- | ---- | --------- | -----------
environment | String | Yes | Environment name
project | String | Yes | Project name
client | String | Yes | Client name
version | String | Yes | Deployed version
alive | Boolean | No, default `false` | Is still alive.
inProgress | Boolean | No, default `false` | Is in progress.

### Delete progress of a deployment

> delete a deployment

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --request DELETE                             \
  --data 'DATA_TO_SEND'                        \
  http://spy:8080/octo-spy/api/deployments/progress
```

> Example of data to send:

```json
{
  "environment": "Production",
  "project": "octo-spy",
  "client": "SomeClient",
  "version": "1.0.0",
}
```

* Need to authentication: **Yes**
* Token allowed: **Yes**
* Path: `/octo-spy/api/deployments/progress`
* Method: `DELETE`
* Data type: `JSON`
* Success status: `204 - No content`
* Errors status:
  * `400 - Field value is empty`: On blank environment or project
  * `404 - Entity not found`: On unknown environment, project, deployment or deploymentProgress

<aside class="notice">
Data is used to identify progress deployment to delete.
</aside>

Available body parameters:

Field name | Search type | Mandatory
---------- | ----------- | ---------
id | NUMBER | No
projectId | NUMBER | No
environment | TEXT | Yes
project | TEXT | Yes
masterProject | TEXT | No
client | TEXT | No
version | TEXT | No
alive | BOOLEAN | No
inProgress | BOOLEAN | No

## Environment

### Get all environments

> Get all application environments

```bash
curl                                  \
  --request GET                       \
  http://spy:8080/octo-spy/api/environments
```

> Example of success:

```json
[
  {
    "id": 1,
    "name": "Production",
    "position": 0
  }
]
```

* Need to authentication: No
* Path: `/octo-spy/api/clients`
* Method: `GET`
* Success status: `200 - Ok`

Key | Type | Description
--- | ---- | -----------
id | Number | Primary key
name | String | Environment name

### Create an environment

> Create an environment

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --header "Content-Type: application/json"    \
  --request POST                               \
  --data 'DATA_TO_SEND'                        \
  http://spy:8080/octo-spy/api/environments
```

> Example of data to send:

```json
{
  "name": "Production",
  "position": 1
}
```

> Example of success:

```json
{
  "id": 1,
  "name": "Production",
  "position": 1
}
```

* Need to authentication: **Yes**
* Token allowed: **Yes**
* Path: `/octo-spy/api/environments`
* Method: `POST`
* Data type: `JSON`
* Success status: `201 - Created`
* Errors status:
  * `400 - Field value is empty`: On blank name
  * `400 - Wrong field value.`: On duplicate name

Body parameters:

Key | Type | Mandatory | Description
--- | ---- | --------- | -----------
name | String | Yes | Environment name
position | Integer | No | Order of environment

### Update an environment

> Update an environment

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --header "Content-Type: application/json"    \
  --request PATCH                              \
  --data 'DATA_TO_SEND'                        \
  http://spy:8080/octo-spy/api/environments/[id]
```

> Example of data to send:

```json
{
  "name": "Production",
  "position": 1
}
```

* Need to authentication: **Yes**
* Token allowed: **Yes**
* Path: `/octo-spy/api/environments/[id]`
* Method: `PATCH`
* Query parameter:
  * `id`: id of environment
* Data type: `JSON`
* Success status: `204 - No content`
* Errors status:
  * `400 - Field value is empty`: On blank name
  * `400 - Wrong field value.`: On duplicate name
  * `404 - Entity not found`: On unknown environment

Body parameters:

Key | Type | Mandatory | Description
--- | ---- | --------- | -----------
name | String | No | Environment name
position | Integer | No | Order of environment

### Delete an environment

> delete an environment

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --request DELETE                             \
  --data 'DATA_TO_SEND'                        \
  http://spy:8080/octo-spy/api/environments/[id]
```

> Example of data to send:

```json
{
  "environment": "Production",
  "project": "octo-spy",
  "client": "SomeClient",
  "version": "1.0.0",
}
```

* Need to authentication: **Yes**
* Token allowed: **Yes**
* Path: `/octo-spy/api/environments`
* Method: `DELETE`
* Query parameter:
  * `id`: id of environment
* Data type: `JSON`
* Success status: `204 - No content`
* Errors status:
  * `404 - Entity not found`: On unknown id

## Application information

> Get application information

```bash
curl                                  \
  --request GET                       \
  http://spy:8080/octo-spy/api/info
```

> Example of success:

```json
{
  "project": null,
  "version": null,
  "environment": null,
  "client": null
}
```

* Need to authentication: No
* Path: `/octo-spy/api/info`
* Method: `GET`
* Success status: `200 - Ok`

Key | Type | Description
--- | ---- | -----------
environment | String | Environment name
project | String | Project name
client | String | Client name
version | String | Deployed version

## Project

### Get project by id

> Get project by id

```bash
curl                                  \
  --request GET                       \
  http://spy:8080/octo-spy/api/projects/[id]
```

> Example of success:

```json
{
  "id": 0,
  "name": null,
  "insertDate": null,
  "updateDate": null
}
```

* Need to authentication: No
* Path: `/octo-spy/api/projects/[id]`
* Method: `GET`
* Query parameter:
  * `id`: id of project
* Success status: `200 - Ok`
* Errors status:
  * `400 - Field value is empty`: On blank id
  * `404 - Entity not found`: On unknown id

Key | Type | Description
--- | ---- | -----------
id | Number | Primary key
name | String | Project name
insertDate | Date | Creation date, format: `yyyy/MM/dd HH:mm:ss`
updateDate | Date | Last update date, format: `yyyy/MM/dd HH:mm:ss`

### Get all projects

> Get all projects

```bash
curl                                  \
  --request GET                       \
  http://spy:8080/octo-spy/api/projects
```

> Example of success:

```json

{
  "total": 1,
  "page": 0,
  "count": 1,
  "resources": [{
    "id": 0,
    "name": null,
    "insertDate": null,
    "updateDate": null
  }]
}
```

* Need to authentication: No
* Path: `/octo-spy/api/projects`
* Method: `GET`
* Success status:
  * `200 - Ok`: When all resources are returned
  * `206 - Partial content`: When partial resources are returned

Available query parameters:

Field name | Search type
---------- | -----------
id | NUMBER
name | TEXT

### Update a project

> Update a project

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --header "Content-Type: application/json"    \
  --request PATCH                              \
  --data 'DATA_TO_SEND'                        \
  http://spy:8080/octo-spy/api/projects/[id]
```

> Example of data to send:

```json
{
  "name": "octo",
  "color": "12,23,199"
}
```

* Need to authentication: **Yes**
* Token allowed: No
* Path: `/octo-spy/api/projects`
* Method: `PATCH`
* Data type: `JSON`
* Success status: `204 - No content`
* Errors status:
  * `400 - Field value is empty`: On blank id
  * `404 - Entity not found`: On unknown id

Body parameters:

Key | Type | Mandatory | Description
--- | ---- | --------- | -----------
name | String | No | Project name
color | String | No | Color of project, format 'R,G,B' where R/G/B is insteger between 0 and 255

### Create a project

> Create a project

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --header "Content-Type: application/json"    \
  --request POST                               \
  --data 'DATA_TO_SEND'                        \
  http://spy:8080/octo-spy/api/projects
```

> Example of data to send:

```json
{
  "name": "octo",
  "isMaster": true,
  "masterName": null,
}
```

> Example of success:

```json
{
  "id": 1,
  "name": "octo",
  "insertDate": null,
  "updateDate": null
}
```

* Need to authentication: **Yes**
* Token allowed: No
* Path: `/octo-spy/api/projects`
* Method: `POST`
* Data type: `JSON`
* Success status: `201 - Created`
* Errors status:
  * `400 - Field value is empty`: On blank name
  * `404 - Entity not found`: On unknown master project name

Body parameters:

Key | Type | Mandatory | Description
--- | ---- | --------- | -----------
name | String | Yes | Project name
isMaster | Boolean | No, default `false` | Is master project
masterName | String | No | Master project name

<aside class="notice">
If <code>isMaster</code> is set to <code>true</code>, you cannot set masterName.
</aside>

### Delete a project

> delete a project

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --request DELETE                             \
  http://spy:8080/octo-spy/api/projects/[id]
```

* Need to authentication: **Yes**
* Token allowed: No
* Path: `/octo-spy/api/projects/[id]`
* Method: `DELETE`
* Query parameter:
  * `id`: id of project
* Success status: `204 - No content`
* Errors status:
  * `400 - Field value is empty`: On blank id
  * `404 - Entity not found`: On unknown id

## Report

### Deployments

> Deployments report

```bash
curl                                  \
  --request GET                       \
  http://spy:8080/octo-spy/api/report/deployments
```

> Example of success:

```json
[{
  "id": 0,
  "masterProject": null,
  "project": null,
  "environment": null,
  "client": null,
  "year": 0,
  "month": 0,
  "dayOfWeek": 0,
  "day": 0,
  "hour": 0,
  "count": 0
}]
```

* Need to authentication: No
* Path: `/octo-spy/api/report/deployments`
* Method: `GET`
* Success status:
  * `200 - Ok`: When all resources are returned

Available query parameters:

Field name | Search type
---------- | -----------
id | NUMBER
masterProject | NUMBER
project | NUMBER
environment | NUMBER
client | TEXT
year | NUMBER
month | NUMBER
dayOfWeek | NUMBER
day | NUMBER
hour | NUMBER

<aside class="notice">
<code>fields</code> is used to select wanted attributes in response.
</aside>

## User

### Get my information

> Get my information

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --request GET                                \
  http://spy:8080/octo-spy/api/users/me
```

> Example of success:

```json
{
  "user": {
    "login": null,
    "firstname": null,
    "lastname": null,
    "email": null,
    "active": true,
    "insertDate": null,
    "updateDate": null
  },
  "roles": []
}
```

* Need to authentication: **Yes**
* Token allowed: **Yes**
* Path: `/octo-spy/api/users/me`
* Method: `GET`
* Success status: `200 - Ok`

Key | Type | Description
--- | ---- | -----------
user.login | String | Login
user.firstname | String | First name
user.lastname | String | Last name
user.email | String | E-mail
user.active | String | Is active
user.insertDate | Date | Creation date, format: `yyyy/MM/dd HH:mm:ss`
user.updateDate | Date | Last update date, format: `yyyy/MM/dd HH:mm:ss`
roles | Array | List of user role

### Create a user token

> Create a user token

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --header "Content-Type: application/json"    \
  --request POST                               \
  --data 'DATA_TO_SEND'                        \
  http://spy:8080/octo-spy/api/users/token
```

> Example of data to send:

```json
TOKEN_NAME
```

> Example of success:

```json
{
  "token": "generated token value"
}
```

* Need to authentication: **Yes**
* Token allowed: No
* Path: `/octo-spy/api/users/token`
* Method: `POST`
* Data type: `JSON`
* Success status: `201 - Created`
* Errors status:
  * `400 - Wrong field value`: On duplicate token name for an user

Body parameters: Token name.

### Delete a user token

> delete a user token

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --request DELETE                             \
  http://spy:8080/octo-spy/api/users/token/[name]
```

* Need to authentication: **Yes**
* Token allowed: No
* Path: `/octo-spy/api/users/token/[name]`
* Method: `DELETE`
* Query parameter:
  * `name`: token name
* Success status: `204 - No content`
* Errors status:
  * `400 - Field value is empty`: On blank name
  * `404 - Entity not found`: On unknown token for user

### Get all token's names of a user

> Get all token's names of a user

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --request GET                                \
  http://spy:8080/octo-spy/api/users/token
```

> Example of success:

```json
["token1", "token2"]
```

* Need to authentication: Yes
* Token allowed: No
* Path: `/octo-spy/api/users/token`
* Method: `GET`
* Success status:
  * `200 - Ok`: When all resources are returned

# Future developments

## New Features

* LDAP authentication, imply this features:
  * LDAP settings page
  * Managing user (enable/disable and add new user)
  * Managing user roles
  * Notification center for user. Send mail on wanted deployment.
* Add explanation/schema on models
* Add "auto-refresh" fonctionnality
* Add information on project/deployment card like :
  * Jira
  * Sonar
  * ssh

And many more!

## API refacto

* Standardize all endpoint return

# Contribution

Octo community is growing incredibly fast and if you’re reading this, there’s a good chance you’re ready to join it. So… welcome!

To improve Octo's documentation make a pull request on this [github repository](https://github.com/Zorin95670/octo-docs).

To improve Octo's API make a pull request on this [github repository](https://github.com/Zorin95670/octo-spy).

To improve Octo's web application make a pull request on this [github repository](https://github.com/Zorin95670/octo-board).
