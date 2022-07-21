# Sequelize and Database Setup

We’ll start by installing the following dependencies. Make sure your terminal or cmd is currently on your project root directory. Then run the following commands:

```
npm install -g sequelize
npm install --save sequelize sequelize-typescript pg-hstore pg
npm install --save-dev @types/sequelize
npm install dotenv --save
```

Now, create a database module.

Run

```
nest generate module /core/database.
```

## Database Interface

Inside the database folder, create an `interfaces` folder, then create a `dbConfig.interface.ts` file inside it. This is for the database configuration interface.

Each of the database environments should optionally have the following properties. Copy and paste the following code:

```typescript
export interface IDatabaseConfigAttributes {
  username?: string;
  password?: string;
  database?: string;
  host?: string;
  port?: number | string;
  dialect?: string;
  urlDatabase?: string;
}

export interface IDatabaseConfig {
  development: IDatabaseConfigAttributes;
  test: IDatabaseConfigAttributes;
  production: IDatabaseConfigAttributes;
}
```

<sup>`src/core/database/interfaces/dbConfig.interface.ts`</sup>

## Database Configuration

Now, let’s create a database environment configuration. Inside the database folder, create a `database.config.ts` file. Copy and paste the below code:

```typescript
import * as dotenv from "dotenv";
import { IDatabaseConfig } from "./interfaces/dbConfig.interface";

dotenv.config();

export const databaseConfig: IDatabaseConfig = {
  development: {
    username: process.env.DB_USER,
    password: process.env.DB_PASS,
    database: process.env.DB_NAME_DEVELOPMENT,
    host: process.env.DB_HOST,
    port: process.env.DB_PORT,
    dialect: process.env.DB_DIALECT,
  },
  test: {
    username: process.env.DB_USER,
    password: process.env.DB_PASS,
    database: process.env.DB_NAME_TEST,
    host: process.env.DB_HOST,
    port: process.env.DB_PORT,
    dialect: process.env.DB_DIALECT,
  },
  production: {
    username: process.env.DB_USER,
    password: process.env.DB_PASS,
    database: process.env.DB_NAME_PRODUCTION,
    host: process.env.DB_HOST,
    dialect: process.env.DB_DIALECT,
  },
};
```

<sup>`src/core/database/database.config.ts`</sup>

The environment will determine which configuration should be used.

## .env file

On our project root folder, create `.env` and `.env.sample` files. Copy and paste the following code into the files:

```typescript
DB_HOST=localhost
DB_PORT=5432
DB_USER=database_user_name
DB_PASS=database_password
DB_DIALECT=postgres
DB_NAME_TEST=test_database_name
DB_NAME_DEVELOPMENT=development_database_name
DB_NAME_PRODUCTION=production_database_name
JWTKEY=random_secret_key
TOKEN_EXPIRATION=48h
BEARER=Bearer
```

<sup>`.env.sample`</sup>

```typescript
DB_HOST=localhost
DB_PORT=5438
DB_USER=blogger
DB_PASS=
DB_DIALECT=postgres
DB_NAME_TEST=blog_test
DB_NAME_DEVELOPMENT=blog_dev
DB_NAME_PRODUCTION=blog_prod
JWTKEY=random_secret_key
TOKEN_EXPIRATION=48h
BEARER=Bearer
```

<sup>`.env`</sup>

Fill the values with the correct information – only on the `.env` file – and make sure it’s added to the `.gitignore` file to avoid pushing it online. The .env.sample is for those who want to download your project and use it so you can push it online.

> **HINTS**: Your username, password, and database name should be what you use to set up your Postgres in the `docker-compose.yaml` file. Which created a Postgres database with your database name.

Nest provides a `@nestjs/config` package out-of-the-box to help load our .env file. To use it, we first install the required dependency.

Run

```bash
npm i --save @nestjs/config.
```

Import the `@nestjs/config` into your app root module:

```typescript
import { Module } from "@nestjs/common";
import { ConfigModule } from "@nestjs/config";

@Module({
  imports: [ConfigModule.forRoot({ isGlobal: true })],
})
export class AppModule {}
```

<sup>`src/app.module.ts`</sup>

Setting the `ConfigModule.forRoot({ isGlobal: true })` to `isGlobal: true` will make the .env properties available throughout the application.

## Database Provider

Let’s create a database provider. Inside the `database` folder, create a file called `database.providers.ts.`

The `core` directory will contain all our core setups, configuration, shared modules, pipes, guards, and middlewares.

In the `database.providers.ts` file, copy and paste this code:

```typescript
import { Sequelize } from "sequelize-typescript";
import { SEQUELIZE, DEVELOPMENT, TEST, PRODUCTION } from "../constants";
import { databaseConfig } from "./database.config";

export const databaseProviders = [
  {
    provide: SEQUELIZE,
    useFactory: async () => {
      let config;
      switch (process.env.NODE_ENV) {
        case DEVELOPMENT:
          config = databaseConfig.development;
          break;
        case TEST:
          config = databaseConfig.test;
          break;
        case PRODUCTION:
          config = databaseConfig.production;
          break;
        default:
          config = databaseConfig.development;
      }
      const sequelize = new Sequelize(config);
      sequelize.addModels(["models goes here"]);
      await sequelize.sync();
      return sequelize;
    },
  },
];
```

<sup>`src/core/database/database.providers.ts`</sup>

Here, the application decides what environment we are currently running on and then chooses the environment configuration.

All our models will be added to the `sequelize.addModels([User, Post])` function. Currently, there are no models.

> **Best practice**: It is a good idea to keep all string values in a constant file and export it to avoid misspelling those values. You'll also have a single place to change things.

Inside the `core` folder, create a `constants` folder and inside it create an `index.ts` file. Paste the following code:

```typescript
export const SEQUELIZE = 'SEQUELIZE';
export const DEVELOPMENT = 'development';
export const TEST = 'test';
export const PRODUCTION = 'production';
src/core/constants/index.ts
Let’s add the database provider to our database module. Copy and paste this code:

import { Module } from '@nestjs/common';
import { databaseProviders } from './database.providers';

@Module({
providers: [...databaseProviders],
exports: [...databaseProviders],
})
export class DatabaseModule { }
```

<sup>`src/core/database/database.module.ts`</sup>

We exported the database provider `exports: [...databaseProviders]` to make it accessible to the rest of the application that needs it.

Now, let’s import the database module into our app root module to make it available to all our services.

```typescript
import { Module } from "@nestjs/common";
import { ConfigModule } from "@nestjs/config";
import { DatabaseModule } from "./core/database/database.module";

@Module({
  imports: [ConfigModule.forRoot({ isGlobal: true }), DatabaseModule],
})
export class AppModule {}
```

<sup>`src/app.module.ts`</sup>

Now we have a ready database module and can move on to the [Next Step: Endpoint Prefix](./003%20endpoint-prefix.md)

---

### Further Read

- https://docs.nestjs.com/recipes/sql-sequelize
