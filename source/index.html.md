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
* API: [octo-spy](https://github.com/Zorin95670/octo-spy/tree/1.11.1) made in Java
* Web application: [octo-board](https://github.com/Zorin95670/octo-board/tree/2.6.1) made in Vue-js

## Docker
> Build `octo-spy` and `octo-board` docker images:

```bash
docker build -t octo-spy octo-spy/

docker build -t octo-board octo-board/
```

To use Octo with docker you must build `octo-spy` and `octo-board` images.

Once images build, you can setup your environment like the [compose example](https://github.com/Zorin95670/octo/blob/1.3.0/docker-compose.yml).

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

# Build changelog page
npm run changelog --silent > public/changelog.html

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

> Authentication example to get user information

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --request GET                                \
  http://spy:8080/octo-spy/api/users/me
```

You need to encode in base64 your `user:password`.

## Register master project

> Register octo as master project

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --header "Content-Type: application/json"    \
  --request POST                               \
  --data 'DATA_TO_SEND'                                 \
  http://spy:8080/octo-spy/api/project
```

> Data to send:

```json
{
  "name": "octo",
  "isMaster": true
}
```

A master project is a project that contains multiple sub-projects.

Default dashboard application displays only master projects.

## Register sub-project

> Register octo-spy as sup-project of octo

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --header "Content-Type: application/json"    \
  --request POST                               \
  --data 'DATA_TO_SEND'                        \
  http://spy:8080/octo-spy/api/project
```

> Data to send:

```json
{
  "name": "octo-spy",
  "masterName": "octo"
}
```

A sub-project is a component of master project.

For example, `octo-spy` and `octo-board` are a sub-projects of `octo`.

## Register `in progress` deployment

> Register new `in progress` deployment for octo-spy version 1.0.0 in production for SomeClient

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --header "Content-Type: application/json"    \
  --request POST                               \
  --data 'DATA_TO_SEND'                        \
  http://spy:8080/octo-spy/api/deployment
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
  http://spy:8080/octo-spy/api/deployment/progress
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
value | String | Field value or error explanation
cause | String | Stack trace of the error.

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
  http://spy:8080/octo-spy/api/client
```

> Example of success:

```json
["Client1", "Client2"]
```

* Need to authentication: No
* Path: `/octo-spy/api/client`
* Method: `GET`
* Success status: `200 - Ok`

## Deployment

### Get deployment by id

> Get deployment by id

```bash
curl                                  \
  --request GET                       \
  http://spy:8080/octo-spy/api/deployment/[id]
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
* Path: `/octo-spy/api/deployment/[id]`
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
  http://spy:8080/octo-spy/api/deployment
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
* Path: `/octo-spy/api/deployment`
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
  http://spy:8080/octo-spy/api/deployment/last
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
* Path: `/octo-spy/api/deployment/last`
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
  http://spy:8080/octo-spy/api/deployment
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
* Path: `/octo-spy/api/deployment`
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

### Delete progress of a deployment

> delete a deployment

```bash
curl                                           \
  --header "Authorization: Basic BASE64_TOKEN" \
  --request DELETE                             \
  --data 'DATA_TO_SEND'                        \
  http://spy:8080/octo-spy/api/deployment/progress
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
* Path: `/octo-spy/api/deployment/progress`
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
environment | TEXT | Yes
project | TEXT | Yes
client | TEXT | No
version | TEXT | No

## Environment

### Get all environments

> Get all application environments

```bash
curl                                  \
  --request GET                       \
  http://spy:8080/octo-spy/api/environment
```

> Example of success:

```json
[
  {
    "id": 1,
    "name": "Production"
  }
]
```

* Need to authentication: No
* Path: `/octo-spy/api/client`
* Method: `GET`
* Success status: `200 - Ok`

Key | Type | Description
--- | ---- | -----------
id | Number | Primary key
name | String | Environment name

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
  http://spy:8080/octo-spy/api/project/[id]
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
* Path: `/octo-spy/api/project/[id]`
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
  http://spy:8080/octo-spy/api/project
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
* Path: `/octo-spy/api/project`
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
  http://spy:8080/octo-spy/api/project/[id]
```

> Example of data to send:

```json
{
  "name": "octo",
  "color": "12,23,199"
}
```

* Need to authentication: **Yes**
* Path: `/octo-spy/api/project`
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
  http://spy:8080/octo-spy/api/project
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
* Path: `/octo-spy/api/project`
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
  http://spy:8080/octo-spy/api/project/[id]
```

* Need to authentication: **Yes**
* Path: `/octo-spy/api/project/[id]`
* Method: `DELETE`
* Query parameter:
  * `id`: id of project
* Success status: `204 - No content`
* Errors status:
  * `400 - Field value is empty`: On blank id
  * `404 - Entity not found`: On unknown id

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

# Future developments

## New Features

* Generate token to avoid using administrator password (include new authentication type)
* Add, update and delete environment form
* Delete deployment in historic page
* Create/delete project in administration page
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
* Standardize all endpoint name, example: `project` -> `projects`

# Contribution

Octo community is growing incredibly fast and if you’re reading this, there’s a good chance you’re ready to join it. So… welcome!

To improve Octo's documentation make a pull request on this [github repository](https://github.com/Zorin95670/octo-docs).

To improve Octo's API make a pull request on this [github repository](https://github.com/Zorin95670/octo-spy).

To improve Octo's web application make a pull request on this [github repository](https://github.com/Zorin95670/octo-board).
