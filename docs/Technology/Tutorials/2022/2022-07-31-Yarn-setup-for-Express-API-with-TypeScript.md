# Yarn setup for Express API with TypeScript

This article talks through the process of setting up a basic Express API using TypeScript and Yarn for package management instead of NPM.

In the folder you'd like to create your project in, type the following commands. For the purpose of this project, we'll assume the folder being used is called “api”.

```powershell
mkdir api
cd api
yarn init --yes
```

This sets up the initial project so that additional packages including Express can be installed.

Next, we'll add the Express and DotEnv packages:

```powershell
yarn add express dotenv
```

At this point, we could test things but we want a typescript app so next we'll add the required TypeScript files as dev dependencies using `-D`:

```powershell
yarn add -D typescript @types/express @types/node
```

The next step is to generate a `tsconfig.json` file which is accomplished by running the following command:

```powershell
yarn tsc --init
```

In `tsconfig.json`, locate the commented out line `// "outDir": "./",` and uncomment it and set the path to `./dist` so the line now looks like the following (note the comment at the end of the line can be left in if desired):

```
"outDir": "./dist",
```

Next create a file called `index.ts` in the root of the project and populate it with the following example code:

```typescript
import express, { Express, Request, Response } from 'express';
import dotenv from 'dotenv';

dotenv.config();

const app: Express = express();
const port = process.env.PORT;

app.get('/', (req: Request, res: Response) => {
  res.send('Express + TypeScript Server');
});

app.listen(port, () => {
  console.log(`⚡️[server]: Server is running at http://localhost:${port}`);
});
```

To make development easier, we'll add a few tools as dev dependencies:

```powershell
yarn add -D concurrently nodemon
```

Next, add or replace the scripts section of package.json with the following (I suggest before the dependencies section):

```json
  "scripts": {
    "build": "npx tsc",
    "start": "node dist/index.js",
    "dev": "concurrently \"npx tsc --watch\" \"nodemon -q dist/index.js\""
  },
```

And update the value of `main` from `index.js`

Finally, create a .env file in the root of the project and populate it with the following (updating the port if required):

```
PORT=3100
```

Finally, start the server in dev mode by typing the following:

```powershell
yarn dev
```

A link to the site will be showing. Using the port suggestion above, this will be [http://localhost:3100/](http://localhost:3100/) which when accessed from the browser will show an empty screen displaying the message “Express + TypeScript Server”.