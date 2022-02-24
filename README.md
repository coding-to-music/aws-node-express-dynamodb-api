<!--
title: 'Serverless Framework Node Express API service backed by DynamoDB on AWS'
description: 'This template demonstrates how to develop and deploy a simple Node Express API service backed by DynamoDB running on AWS Lambda using the traditional Serverless Framework.'
layout: Doc
framework: v3
platform: AWS
language: nodeJS
priority: 1
authorLink: 'https://github.com/serverless'
authorName: 'Serverless, inc.'
authorAvatar: 'https://avatars1.githubusercontent.com/u/13742415?s=200&v=4'
-->

# Serverless Framework Node Express API on AWS

This template demonstrates how to develop and deploy a simple Node Express API service, backed by DynamoDB database, running on AWS Lambda using the traditional Serverless Framework.

## Anatomy of the template

This template configures a single function, `api`, which is responsible for handling all incoming requests thanks to the `httpApi` event. To learn more about `httpApi` event configuration options, please refer to [httpApi event docs](https://www.serverless.com/framework/docs/providers/aws/events/http-api/). As the event is configured in a way to accept all incoming requests, `express` framework is responsible for routing and handling requests internally. Implementation takes advantage of `serverless-http` package, which allows you to wrap existing `express` applications. To learn more about `serverless-http`, please refer to corresponding [GitHub repository](https://github.com/dougmoscrop/serverless-http). Additionally, it also handles provisioning of a DynamoDB database that is used for storing data about users. The `express` application exposes two endpoints, `POST /users` and `GET /user/{userId}`, which allow to create and retrieve users.

## Usage

### Deployment

Install dependencies with:

```
npm install
```

and then deploy with:

```
serverless deploy
```

After running deploy, you should see output similar to:

```bash
Deploying aws-node-express-dynamodb-api-project to stage dev (us-east-1)

✔ Service deployed to stack aws-node-express-dynamodb-api-project-dev (196s)

endpoint: ANY - https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com
functions:
  api: aws-node-express-dynamodb-api-project-dev-api (766 kB)
```

_Note_: In current form, after deployment, your API is public and can be invoked by anyone. For production deployments, you might want to configure an authorizer. For details on how to do that, refer to [`httpApi` event docs](https://www.serverless.com/framework/docs/providers/aws/events/http-api/). Additionally, in current configuration, the DynamoDB table will be removed when running `serverless remove`. To retain the DynamoDB table even after removal of the stack, add `DeletionPolicy: Retain` to its resource definition.

### Invocation

After successful deployment, you can create a new user by calling the corresponding endpoint:

```bash
curl --request POST 'https://xxxxxx.execute-api.us-east-1.amazonaws.com/users' --header 'Content-Type: application/json' --data-raw '{"name": "John", "userId": "someUserId"}'
```

Deployed URL:

https://md6hckc0j4.execute-api.us-east-1.amazonaws.com

```bash
curl --request POST 'https://md6hckc0j4.execute-api.us-east-1.amazonaws.com/users' --header 'Content-Type: application/json' --data-raw '{"name": "John", "userId": "someUserId"}'
```

Which should result in the following response:

```bash
{"userId":"someUserId","name":"John"}
```

You can later retrieve the user by `userId` by calling the following endpoint:

```bash
curl https://xxxxxxx.execute-api.us-east-1.amazonaws.com/users/someUserId
```

curl https://md6hckc0j4.execute-api.us-east-1.amazonaws.com/users/someUserId

Which should result in the following response:

```bash
{"userId":"someUserId","name":"John"}
```

If you try to retrieve user that does not exist, you should receive the following response:

```bash
{"error":"Could not find user with provided \"userId\""}
```

### Local development

It is also possible to emulate DynamoDB, API Gateway and Lambda locally using the `serverless-dynamodb-local` and `serverless-offline` plugins. In order to do that, run:

```bash
serverless plugin install -n serverless-dynamodb-local
serverless plugin install -n serverless-offline
```

It will add both plugins to `devDependencies` in `package.json` file as well as will add it to `plugins` in `serverless.yml`. Make sure that `serverless-offline` is listed as last plugin in `plugins` section:

```
plugins:
  - serverless-dynamodb-local
  - serverless-offline
```

You should also add the following config to `custom` section in `serverless.yml`:

```
custom:
  (...)
  dynamodb:
    start:
      migrate: true
    stages:
      - dev
```

Additionally, we need to reconfigure `AWS.DynamoDB.DocumentClient` to connect to our local instance of DynamoDB. We can take advantage of `IS_OFFLINE` environment variable set by `serverless-offline` plugin and replace:

```javascript
const dynamoDbClient = new AWS.DynamoDB.DocumentClient();
```

with the following:

```javascript
const dynamoDbClientParams = {};
if (process.env.IS_OFFLINE) {
  dynamoDbClientParams.region = "localhost";
  dynamoDbClientParams.endpoint = "http://localhost:8000";
}
const dynamoDbClient = new AWS.DynamoDB.DocumentClient(dynamoDbClientParams);
```

After that, running the following command with start both local API Gateway emulator as well as local instance of emulated DynamoDB:

```bash
serverless offline start
```

To learn more about the capabilities of `serverless-offline` and `serverless-dynamodb-local`, please refer to their corresponding GitHub repositories:

- https://github.com/dherault/serverless-offline
- https://github.com/99x/serverless-dynamodb-local

# serverless-dynamodb-local

[![Join the chat at https://gitter.im/99xt/serverless-dynamodb-local](https://badges.gitter.im/99xt/serverless-dynamodb-local.svg)](https://gitter.im/99xt/serverless-dynamodb-local?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![npm version](https://badge.fury.io/js/serverless-dynamodb-local.svg)](https://badge.fury.io/js/serverless-dynamodb-local)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## This Plugin Requires

- serverless@^1
- Java Runtime Engine (JRE) version 6.x or newer _OR_ docker CLI client

## Features

- Install DynamoDB Local Java program
- Run DynamoDB Local as Java program on the local host or in docker container
- Start DynamoDB Local with all the parameters supported (e.g port, inMemory, sharedDb)
- Table Creation for DynamoDB Local

## Install Plugin

`npm install --save serverless-dynamodb-local`

Then in `serverless.yml` add following entry to the plugins array: `serverless-dynamodb-local`

```yml
plugins:
  - serverless-dynamodb-local
```

## Using the Plugin

1. Install DynamoDB Local (unless using docker setup, see below)
   `sls dynamodb install`

2. Add DynamoDB Resource definitions to your Serverless configuration, as defined here: https://serverless.com/framework/docs/providers/aws/guide/resources/#configuration

3. Start DynamoDB Local and migrate (DynamoDB will process incoming requests until you stop it. To stop DynamoDB, type Ctrl+C in the command prompt window). Make sure above command is executed before this.
   `sls dynamodb start --migrate`

Note: Read the detailed section for more information on advanced options and configurations. Open a browser and go to the url http://localhost:8000/shell to access the web shell for dynamodb local.

## Install: sls dynamodb install

This installs the Java program locally. If using docker, this step is not required.

To remove the installed dynamodb local, run:
`sls dynamodb remove`
Note: This is useful if the sls dynamodb install failed in between to completely remove and install a new copy of DynamoDB local.

## Start: sls dynamodb start

This starts the DynamoDB Local instance, either as a local Java program or, if the `--docker` flag is set,
by running it within a docker container. The default is to run it as a local Java program.

All CLI options are optional:

```
--port  		  -p  Port to listen on. Default: 8000
--cors                    -c  Enable CORS support (cross-origin resource sharing) for JavaScript. You must provide a comma-separated "allow" list of specific domains. The default setting for -cors is an asterisk (*), which allows public access.
--inMemory                -i  DynamoDB; will run in memory, instead of using a database file. When you stop DynamoDB;, none of the data will be saved. Note that you cannot specify both -dbPath and -inMemory at once.
--dbPath                  -d  The directory where DynamoDB will write its database file. If you do not specify this option, the file will be written to the current directory. Note that you cannot specify both -dbPath and -inMemory at once. For the path, current working directory is <projectroot>/node_modules/serverless-dynamodb-local/dynamob. For example to create <projectroot>/node_modules/serverless-dynamodb-local/dynamob/<mypath> you should specify -d <mypath>/ or --dbPath <mypath>/ with a forwardslash at the end.
--sharedDb                -h  DynamoDB will use a single database file, instead of using separate files for each credential and region. If you specify -sharedDb, all DynamoDB clients will interact with the same set of tables regardless of their region and credential configuration.
--delayTransientStatuses  -t  Causes DynamoDB to introduce delays for certain operations. DynamoDB can perform some tasks almost instantaneously, such as create/update/delete operations on tables and indexes; however, the actual DynamoDB service requires more time for these tasks. Setting this parameter helps DynamoDB simulate the behavior of the Amazon DynamoDB web service more closely. (Currently, this parameter introduces delays only for global secondary indexes that are in either CREATING or DELETING status.)
--optimizeDbBeforeStartup -o  Optimizes the underlying database tables before starting up DynamoDB on your computer. You must also specify -dbPath when you use this parameter.
--migration               -m  After starting dynamodb local, run dynamodb migrations.
--heapInitial                 The initial heap size
--heapMax                     The maximum heap size
--migrate                 -m  After starting DynamoDB local, create DynamoDB tables from the Serverless configuration.
--seed                    -s  After starting and migrating dynamodb local, injects seed data into your tables. The --seed option determines which data categories to onload.
--convertEmptyValues      -e  Set to true if you would like the document client to convert empty values (0-length strings, binary buffers, and sets) to be converted to NULL types when persisting to DynamoDB.
--docker                      Run DynamoDB inside docker container instead of as a local Java program
--dockerImage                 Specify custom docker image. Default: amazon/dynamodb-local
```

All the above options can be added to serverless.yml to set default configuration: e.g.

```yml
custom:
  dynamodb:
    # If you only want to use DynamoDB Local in some stages, declare them here
    stages:
      - dev
    start:
      port: 8000
      inMemory: true
      heapInitial: 200m
      heapMax: 1g
      migrate: true
      seed: true
      convertEmptyValues: true
    # Uncomment only if you already have a DynamoDB running locally
    # noStart: true
```

Docker setup:

```yml
custom:
  dynamodb:
    # If you only want to use DynamoDB Local in some stages, declare them here
    stages:
      - dev
    start:
      docker: true
      port: 8000
      inMemory: true
      migrate: true
      seed: true
      convertEmptyValues: true
    # Uncomment only if you already have a DynamoDB running locally
    # noStart: true
```

## Migrations: sls dynamodb migrate

### Configuration

In `serverless.yml` add following to execute all the migration upon DynamoDB Local Start

```yml
custom:
  dynamodb:
    start:
      migrate: true
```

### AWS::DynamoDB::Table Resource Template for serverless.yml

```yml
resources:
  Resources:
    usersTable:
      Type: AWS::DynamoDB::Table
      Properties:
        TableName: usersTable
        AttributeDefinitions:
          - AttributeName: email
            AttributeType: S
        KeySchema:
          - AttributeName: email
            KeyType: HASH
        ProvisionedThroughput:
          ReadCapacityUnits: 1
          WriteCapacityUnits: 1
```

**Note:**
DynamoDB local doesn't support TTL specification, therefore plugin will simply ignore ttl configuration from Cloudformation template.

## Seeding: sls dynamodb seed

### Configuration

In `serverless.yml` seeding categories are defined under `dynamodb.seed`.

If `dynamodb.start.seed` is true, then seeding is performed after table migrations.

If you wish to use raw AWS AttributeValues to specify your seed data instead of Javascript types then simply change the variable of any such json files from `sources:` to `rawsources:`.

```yml
custom:
  dynamodb:
    start:
      seed: true

    seed:
      domain:
        sources:
          - table: domain-widgets
            sources: [./domainWidgets.json]
          - table: domain-fidgets
            sources: [./domainFidgets.json]
      test:
        sources:
          - table: users
            rawsources: [./fake-test-users.json]
          - table: subscriptions
            sources: [./fake-test-subscriptions.json]
```

```bash
> sls dynamodb seed --seed=domain,test
> sls dynamodb start --seed=domain,test
```

If seed config is set to true, your configuration will be seeded automatically on startup. You can also put the seed to false to prevent initial seeding to use manual seeding via cli.

```fake-test-users.json example
[
  {
    "id": "John",
    "name": "Doe",
  },
]
```

## Using DynamoDB Local in your code

You need to add the following parameters to the AWS NODE SDK dynamodb constructor

e.g. for dynamodb document client sdk

```
var AWS = require('aws-sdk');
```

```
new AWS.DynamoDB.DocumentClient({
    region: 'localhost',
    endpoint: 'http://localhost:8000',
    accessKeyId: 'DEFAULT_ACCESS_KEY',  // needed if you don't have aws credentials at all in env
    secretAccessKey: 'DEFAULT_SECRET' // needed if you don't have aws credentials at all in env
})
```

e.g. for dynamodb document client sdk

```
new AWS.DynamoDB({
    region: 'localhost',
    endpoint: 'http://localhost:8000',
    accessKeyId: 'DEFAULT_ACCESS_KEY',  // needed if you don't have aws credentials at all in env
    secretAccessKey: 'DEFAULT_SECRET' // needed if you don't have aws credentials at all in env

})
```

### Using with serverless-offline plugin

When using this plugin with serverless-offline, it is difficult to use above syntax since the code should use DynamoDB Local for development, and use DynamoDB Online after provisioning in AWS. Therefore we suggest you to use [serverless-dynamodb-client](https://github.com/99xt/serverless-dynamodb-client) plugin in your code.

The `serverless dynamodb start` command can be triggered automatically when using `serverless-offline` plugin.
Please note that you still need to install DynamoDB Local first.

Add both plugins to your `serverless.yml` file:

```yaml
plugins:
  - serverless-dynamodb-local
  - serverless-offline
```

Make sure that `serverless-dynamodb-local` is above `serverless-offline` so it will be loaded earlier.

Now your local DynamoDB database will be automatically started before running `serverless offline`.

### Using with serverless-offline and serverless-webpack plugin

Run `serverless offline start`. In comparison with `serverless offline`, the `start` command will fire an `init` and a `end` lifecycle hook which is needed for serverless-offline and serverless-dynamodb-local to switch off both ressources.

Add plugins to your `serverless.yml` file:

```yaml
plugins:
  - serverless-webpack
  - serverless-dynamodb-local
  - serverless-offline #serverless-offline needs to be last in the list
```

## Reference Project

- [serverless-react-boilerplate](https://github.com/99xt/serverless-react-boilerplate)

## Links

- [Dynamodb local documentation](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.html)
- [Contact Us](mailto:ashanf@99x.lk)
- [NPM Registry](https://www.npmjs.com/package/serverless-dynamodb-local)

## License

[MIT](LICENSE)
