# User Module

Let’s add a User module to handle all user-related operations and to keep tabs on who is creating what post.

Run

```
nest generate module /modules/users.
```

This will automatically add this module to our root module `AppModule`.

## Generate User Service

Run

```
nest generate service /modules/users.
```

This will automatically add this service to the `UsersModule`.

## Set Up User Database Schema Model

Inside `modules/users`, create a file called `user.entity.ts` then copy and paste this code:

```typescript
import { Table, Column, Model, DataType } from "sequelize-typescript";

@Table
export class User extends Model<User> {
  @Column({
    type: DataType.STRING,
    allowNull: false,
  })
  name: string;

  @Column({
    type: DataType.STRING,
    unique: true,
    allowNull: false,
  })
  email: string;

  @Column({
    type: DataType.STRING,
    allowNull: false,
  })
  password: string;

  @Column({
    type: DataType.ENUM,
    values: ["male", "female"],
    allowNull: false,
  })
  gender: string;
}
```

<sup>`src/modules/users/user.entity.ts`</sup>

Here, we are specifying what our User table will contain. The `@Column()` decorator provides information about each column in the table. The User table will have name email password and gender as columns. We imported all the Sequelize decorators from `sequelize-typescript`

## User DTO

Let’s create our User DTO (Data Transfer Object) schema. Inside the users folder, create a dto folder. Then create a user.dto.ts file inside it. Paste the following code in:

```typescript
export class UserDto {
readonly name: string;
readonly email: string;
readonly password: string;
readonly gender: string;
}
src/modules/users/dto/user.dto.ts
User Repository provider
Now, create a User Repository provider. Inside the user's folder, create a users.providers.ts file. This provider is used to communicate with the database.

import { User } from './user.entity';
import { USER_REPOSITORY } from '../../core/constants';

export const usersProviders = [{
provide: USER_REPOSITORY,
useValue: User,
}];
```

<sup>`src/modules/users/users.providers.ts`</sup>

Add this export `const USER_REPOSITORY = 'USER_REPOSITORY'`; to the constants `index.ts` file.

Also, add the user provider to the User module. Notice, we added the `UserService` to our exports array. That is because we’ll need it outside of the User Module.

```typescript
import { Module } from "@nestjs/common";
import { UsersService } from "./users.service";
import { usersProviders } from "./users.providers";

@Module({
  providers: [UsersService, ...usersProviders],
  exports: [UsersService],
})
export class UsersModule {}
```

<sup>`src/modules/users/users.module.ts`</sup>

Let’s encapsulate user operations inside the `UsersService`. Copy and paste the following code:

```typescript
import { Injectable, Inject } from "@nestjs/common";
import { User } from "./user.entity";
import { UserDto } from "./dto/user.dto";
import { USER_REPOSITORY } from "../../core/constants";

@Injectable()
export class UsersService {
  constructor(
    @Inject(USER_REPOSITORY) private readonly userRepository: typeof User
  ) {}

  async create(user: UserDto): Promise<User> {
    return await this.userRepository.create<User>(user);
  }

  async findOneByEmail(email: string): Promise<User> {
    return await this.userRepository.findOne<User>({ where: { email } });
  }

  async findOneById(id: number): Promise<User> {
    return await this.userRepository.findOne<User>({ where: { id } });
  }
}
```

<sup>`src/modules/users/users.service.ts`</sup>

Here, we injected the user repository to communicate with the DB.

`create(user: UserDto)` This method creates a new user into the user table and returns the newly created user object.

`findOneByEmail(email: string)` This method is used to look up a user from the user table by email and returns the user.

`findOneById(id: number)` This method is used to look up a user from the user table by the user Id and returns the user.
We will use these methods later.

Lastly, let’s add the User model to the `database.providers.ts` file `sequelize.addModels([User]);`.

```typescript
import { Sequelize } from "sequelize-typescript";
import { SEQUELIZE, DEVELOPMENT, TEST, PRODUCTION } from "../constants";
import { databaseConfig } from "./database.config";
import { User } from "../../modules/users/user.entity";

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
      sequelize.addModels([User]);
      await sequelize.sync();
      return sequelize;
    },
  },
];
```

<sup>`src/core/database/database.providers.ts`</sup>

Now we have a ready `UserModule` and we can move on to the [Next Step: Auth Module](./005%20auth-module.md)
