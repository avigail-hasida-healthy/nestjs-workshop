# Setting a global endpoint prefix

We might want all our API endpoints to start with `api/v1` for different versioning. We don't want to have to add this prefix to all our controllers. Fortunately, Nest provides a way to set a global prefix.

In the `main.ts` file, add `app.setGlobalPrefix('api/v1');`

```typescript
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  // global prefix
  app.setGlobalPrefix("api/v1");
  await app.listen(3000);
}
bootstrap();
```

<sup>`src/main.ts`</sup>

Now we have a ready endpoint prefix and we can move on to the [Next Step: User Module](./004%20user-module.md)
