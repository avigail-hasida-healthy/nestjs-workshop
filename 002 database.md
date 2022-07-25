# Sequelize and Database Setup

We’ll start by installing the following dependencies. Make sure your terminal or cmd is currently on your project root directory. Then run the following commands:

```
npm install -g sequelize
npm install --save sequelize sequelize-typescript sqlite3
npm install --save-dev @types/sequelize
npm install dotenv --save
```

Now, create a database module.

Run

```
nest generate module /core/database
```

## .env file

On our project root folder, create `.env` file. Copy and paste the following code into the files:

```typescript
DB_USER=blogger
DB_PASS=
DB_STORAGE=:memory:
DB_DIALECT=sqlite
DB_NAME=blog_dev
JWTKEY=random_secret_key
TOKEN_EXPIRATION=48h
BEARER=Bearer
```

<sup>`.env`</sup>

Make sure the `.env` is added to the `.gitignore` file to avoid pushing it online.

Nest provides a `@nestjs/config` package out-of-the-box to help load our `.env` file. To use it, we first install the required dependency.

Run

```bash
npm i --save @nestjs/config
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
import * as dotenv from "dotenv";
import { Sequelize } from "sequelize-typescript";
import { Dialect } from "sequelize/types";
import { SEQUELIZE } from "../constants";

dotenv.config();

export const databaseProviders = [
  {
    provide: SEQUELIZE,
    useFactory: async () => {
      const username = process.env.DB_USERNAME;
      const password = process.env.DB_PASSWORD;
      const database = process.env.DB_NAME;
      const storage = process.env.DB_STORAGE;
      const dialect = process.env.DB_DIALECT as Dialect;

      const sequelize = new Sequelize(database, username, password, {
        dialect,
        storage,
      });
      sequelize.addModels([]);
      await sequelize.sync();
      return sequelize;
    },
  },
];
```

<sup>`src/core/database/database.providers.ts`</sup>

Here we initiate `Sequelize` with a local in memory SQLite database.

All our models will be added to the `sequelize.addModels([User, Post])` function. Currently, there are no models.

> **Best practice**: It is a good idea to keep all string values in a constant file and export it to avoid misspelling those values. You'll also have a single place to change things.

Inside the `core` folder, create a `constants` folder and inside it create an `index.ts` file. Paste the following code:

```typescript
export const SEQUELIZE = "SEQUELIZE";
```

<sup>`src/core/constants/index.ts`</sup>

Let’s add the database provider to our database module. Copy and paste this code:

```typescript
import { Module } from "@nestjs/common";
import { databaseProviders } from "./database.providers";

@Module({
  providers: [...databaseProviders],
  exports: [...databaseProviders],
})
export class DatabaseModule {}
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
- https://docs.nestjs.com/techniques/configuration
