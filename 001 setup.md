# Setup

Install the NestJs CLI. Nest comes with an awesome CLI that makes it easy to scaffold a Nest application with ease. In your terminal or cmd run:

```bash
npm i -g @nestjs/cli
```

Now you have Nest installed globally in your machine.

On your terminal or cmd, cd into the directory where you want to create your application, and run following commands:

```bash
nest new nest-blog-api
cd nest-blog-api
npm run start:dev
```

Navigate to http://localhost:3000 on any of your browsers. You should see Hello World. Bravo! you have created your first Nest app. Let’s continue.

> **NOTE**: As of this writing, if running npm run start:dev throws an error, change your typescript:3.4.2 in your package.json file to typescript:3.7.2 and then delete the node_modules and package-lock.json re-run npm i.

Your folder structure should look like this:

![folder structure](https://www.freecodecamp.org/news/content/images/2020/06/1_qLCtw-62xXAdTHXPy2JAMg.png)

Now we have a running nestJs project lets start to build a blog api in the [Next Step: Database](./002%20database.md)
