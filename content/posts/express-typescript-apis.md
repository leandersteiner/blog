---
title: 'Building APIs with Express, MongoDB & Typescript'
date: 2022-05-15
author: 'Leander Steiner'
draft: false
description: 'Using typescript in combination with the express web framework to build apis storing data in mongodb'
---

# Express, MongoDB & Typescript

## Getting everything ready

Whenever we start a new project with Node.js we have to initialize a new npm package with the `npm init` command.
This will create a file called `package.json`, which is used to keep track of any external dependencies we install into the project and allows us to define custom scripts.

### Installing dependencies

#### Dev Dependencies

- eslint: Linter for javascript & typescript
- nodemon: Tool to automatically reload code changes
- rimraf: Tool to delete files & folders
- ts-node: Tool to run typescript code like normal javascript code
- typescript: Typescript compiler
- @types/{package}: Type definitions for imported packages

```
npm install -D eslint nodemon rimraf ts-node typescript @types/express @types/node @typescript-eslint/eslint-plugin @typescript-eslint/parser
```

#### Other

- express: Web framework based on Node.js
- mongodb: Database driver

```
npm install express mongodb
```

### Creating npm scripts

Npm scripts are very handy for any tasks pertaining to actions we need to execute to get our project running or built.
We will create scripts for starting our server, building out project for production, cleaning our directory after building it and for linting. Any scripts we want to define go into our `package.json` file inside the **scripts** object.

```json
"scripts": {
  "start:dev": "nodemon src/index.ts",
  "start": "npm run build && node dist/watchlist/src/index.js",
  "clean": "rimraf ./dist",
  "build": "npm run clean && tsc",
  "lint": "eslint . --ext .ts"
}
```

### Configuring typescript and eslint

Next we want to configure typescript. To generate a default tsconfig.json just run `npx tsc --init`.

tsconfig.js

```json
{
  "compilerOptions": {
    "target": "ES2021",
    "module": "commonjs",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "outDir": "dist",
    "rootDir": "src"
  }
}
```

.eslintrc.json

```json
{
  "env": {
    "browser": false,
    "commonjs": true,
    "es2021": true
  },
  "extends": ["eslint:recommended", "plugin:@typescript-eslint/recommended"],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": "latest"
  },
  "plugins": ["@typescript-eslint"],
  "rules": {}
}
```

.eslintignore

```
node_modules
dist
```

## What we will build

As an example we will build an api that will us allow to retrieve movies and reviews for those movies.

This means we will have two domain models, `Movie` and `Review`.
A movie will consist of a title and a release year, the reviews will belong to a movie and consist of the reviewers name, a rating, a title and the actual review text.

## Building the api

### Folder structure

!["folders.png"]](test)