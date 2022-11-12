# steps to reproduce

1. [Following this guide step by step to initialize repo](https://turbo.build/repo/docs/getting-started/existing-monorepo)
2. Create package.json with following contents

```
{
    "workspaces": [
        "packages/*",
        "apps/*"
    ],
    "private": true,
    "scripts": {
        "build": "turbo run build",
        "test": "turbo run test",
        "lint": "turbo run lint",
        "dev": "turbo run dev"
    }
}
```

3. `yarn install`
4. `yarn add turbo -DW`
5. Create turbo.json with following contents

```
{
   "$schema": "https://turbo.build/schema.json",
    "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**"]
    },
    "test": {
      "dependsOn": ["build"],
      "outputs": [],
      "inputs": ["src/**/*.tsx", "src/**/*.ts", "test/**/*.ts", "test/**/*.tsx"]
    },
    "lint": {
      "outputs": []
    },
    "deploy": {
      "dependsOn": ["build", "test", "lint"],
      "outputs": []
    },
    "dev": {
      "cache": false
    }
  }
}
```

6. Create directories `packages` and `apps`
7. `cd apps`
8. `npx create-next-app@latest` and name it `web`, select typescript and eslint
9. `cd ..`
10. `yarn install`
11. `yarn dev`
12. Everything works smoothly
13. Proceed using [Deploying with Docker guide](https://turbo.build/repo/docs/handbook/deploying-with-docker)
14. Replicate [example Dockerfile](https://turbo.build/repo/docs/handbook/deploying-with-docker#example) to /apps/web/Dockerfile
15. add .gitignore to root with following contents

```
.turbo/**
build/**
dist/**
.next/**
```

16. Add `output: 'standalone'` to next.config.js referring to [this issue](https://stackoverflow.com/questions/70608086/i-am-getting-error-while-converting-my-next-js-project-to-docker)
17. Create docker-compose.yml to root with following contents

```
version: "3"

services:
  web:
    container_name: web
    build:
      context: .
      dockerfile: ./apps/web/Dockerfile
    restart: always
    ports:
      - 3000:3000

```

17. `docker-compose up --build`

# Expected behavior

I expect the app to be running on localhost:3000

# Actual behavior

I get the following error

```
Attaching to web
web    | node:internal/modules/cjs/loader:998
web    |   throw err;
web    |   ^
web    |
web    | Error: Cannot find module '/app/apps/web/server.js'
web    |     at Module._resolveFilename (node:internal/modules/cjs/loader:995:15)
web    |     at Module._load (node:internal/modules/cjs/loader:841:27)
web    |     at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:82:12)
web    |     at node:internal/main/run_main_module:23:47 {
web    |   code: 'MODULE_NOT_FOUND',
web    |   requireStack: []
web    | }
web    |
web    | Node.js v19.0.1
```
