# Swagger

Let's add swagger to keep api documentation
Nest provides a `@nestjs/swagger` package out-of-the-box to help load our all the api documentation into a swagger document. To use it, we first install the required dependency.

Run

```bash
$ npm install --save @nestjs/swagger
```

Once the installation process is complete, open the `main.ts` file and initialize Swagger using the `SwaggerModule`class:

```typescript
import { NestFactory } from "@nestjs/core";
import { SwaggerModule, DocumentBuilder } from "@nestjs/swagger";
import { AppModule } from "./app.module";
import { ValidateInputPipe } from "./core/pipes/validate.pipe";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  // global endpoints prefix
  app.setGlobalPrefix("api/v1");
  // handle all user input validation globally
  app.useGlobalPipes(new ValidateInputPipe());
  // setup swagger
  const config = new DocumentBuilder()
    .setTitle("Blog API")
    .setDescription("The blog API description")
    .setVersion("1.0")
    .build();
  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup("api/docs", app, document);

  await app.listen(3000);
}
bootstrap();
```

<sup>`main.ts`</sup>

Open the `user.dto.ts` file in the `dto` folder and add `@ApiProperty()` to each of the fields. The `@ApiProperty()` decorator notifies swagger the field is a required property in the dto.

```typescript
import { ApiProperty } from "@nestjs/swagger";
import { IsNotEmpty, MinLength, IsEmail, IsEnum } from "class-validator";

enum Gender {
  MALE = "male",
  FEMALE = "female",
}

export class UserDto {
  @ApiProperty()
  @IsNotEmpty()
  readonly name: string;

  @ApiProperty()
  @IsNotEmpty()
  @IsEmail()
  readonly email: string;

  @IsNotEmpty()
  @MinLength(6)
  readonly password: string;

  @ApiProperty()
  @IsNotEmpty()
  @IsEnum(Gender, {
    message: "gender must be either male or female",
  })
  readonly gender: string;
}
```

<sup>`/src/modules/users/dto/user.dto.ts`</sup>

The same should be applied to `post.dto.ts` file.

```typescript
import { ApiProperty } from "@nestjs/swagger";
import { IsNotEmpty, MinLength } from "class-validator";

export class PostDto {
  @ApiProperty()
  @IsNotEmpty()
  @MinLength(4)
  readonly title: string;

  @ApiProperty()
  @IsNotEmpty()
  readonly body: string;
}
```

<sup>`/src/modules/posts/dto/post.dto.ts`</sup>

## Let's see swagger in action...

Open your browser and set the url to `http://localhost:3000/api/docs/`

## ![Swagger](./images/011-swagger.gif)

Beautiful, we have a blog api up and running!

**WELL DONE!**

---

### Further Read

- https://docs.nestjs.com/openapi/introduction
