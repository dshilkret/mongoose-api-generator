## Mongoose REST API Autogenerator

Automatically generate a REST API from Mongoose models.

Creates a hot-reloading server that auto-updates whenever models are updated or created.

:warning: This project is still in an early stage and may undergo breaking changes.

## Dev

Create a `.env` file with

```bash

# url of mongo database
MONGODB_URL='<url_of_mongo_database>'

# jwt signing secret for auth
JWT_SECRET = '<signing_secret>'

# directory models are discovered from (optional)
# ./models by default
MODELS_DIR = 'models'

# the path of the autogenerated resources enum file for the client (optional)
# ./client/src by default
RESOURCES_FILE_DIR = 'client/src'

```

Then:

```bash
yarn install
yarn start
```

- API calls can be made using `curl`, [Httpie](https://httpie.org/), or a full-fledged API client like [Insomnia](https://insomnia.rest/) or [Postman](https://www.postman.com/).

## Client library

This repo includes a sample frontend (React + TypeScript) at `./client`. This is a bare React app made with [create-snowpack-app](https://github.com/pikapkg/snowpack) using the `@snowpack/app-template-react-typescript` template.

There are two custom files included to make API requests easier.

- `client/src/apiLib.ts` - exports helpful functions to easily interact with the API (eg. `signup()`, `login()`, `api.CREATE()`, `api.GET()`, etc.).
- `client/src/apiResources.ts` - contains an exported TypeScript enum made to work with `apiLib.ts` that is autogenerated from your models, and is hot-reloaded as well.

Sample usage:

```js
import { signup, login, api, Resource } from "./apiLib";
await signup("bob", email, password);
await login(email, password);
// create a new box
let box = await api.CREATE(Resource.box, { height: 4 });
// get a new box
box = await api.GET(Resource.box, box._id);
// list all boxes
box = await api.LIST(Resource.box);
// api.UPDATE and api.DELETE is also available.
```

## Authentication

- Sign up:
  - `POST /auth/signup`
  - Sample request body:
    ```
    {"username": "bob", "email": "bob@gmail.com", "password": "badpw"}
    ```
- Login:

  - `POST /auth/login`
  - Sample request body:
    ```
    {"email": "bob@gmail.com", "password": "badpw"}
    ```
    - returns a JWT token, eg. `Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjVlZWQ3OWM1N2MxZmEzNzExODZlZjljOSIsInVzZXJuYW1lIjoiYWIiLCJpYXQiOjE1OTI2MjI1MDAsImV4cCI6MTU5MjY1ODUwMH0.-eFJ1FcotHsjtmgUaE3f-6fFz_7y8c2dCNqhH8E5S6A`
  - Pass the token in the `Authorization` HTTP header for each subsequent API request.

- View user profile via `GET /auth/profile`

## Adding a model

Simply create a new file that exports a mongoose model to `./models`.

This will result in two actions:

1. autogenerated URL endpoints will be updated
2. An autogenerated file will be created/overriden at `${RESOURCES_FILE_DIR}/apiResources.ts`.

**NOTE: these endpoints are not accessible unless you are signed in.**
The autogenerated URL endpoints will be:

Create:

- `POST /api/{fileName}`
  - Inputs are automatically validated using the Mongoose Schema. Errors are returned to the client with a `400` HTTP Error code.

List all

- `GET /api/{fileName}`

Get one

- `GET /api/{fileName}/:id`

Update one

- `PATCH /api/{fileName}/:id`

Delete one

- `DELETE /api/{fileName}/:id`

## Implementing permissions

Permissions enable you to implement granular restrictions on who can perform an action on a resource.

- need to have an `owner_id: String` field in the Mongoose Schema. This field is automatically populated whenever a new object is created via the API endpoint.
- Export a `permissions` object that may override `list/get/update/remove` fields (by default all of these are set to `PUBLIC`)

  - possible values (exported from `"../system/auth/permissions"`):
    - `PUBLIC`: anyone can perform this action on this resource
    - `OWNER`: only the creator of the resource can perform this action
    - `NONE`: this action is disabled for this object

- Example:

```js
const { Schema } = require("mongoose");
const { PUBLIC, OWNER, NONE } = require("../framework/auth/permissions");
const schema = new Schema(
  {
    width: Number,
    height: Number,
    created: { type: Date, default: Date.now },
    name: String,
    owner_id: String,
  },
  { strict: "throw" }
);

const permissions = {
  list: PUBLIC,
  get: PUBLIC,
  update: OWNER,
  remove: NONE,
};

module.exports = { schema, permissions };
```

## Tech used

- Mongoose, Express, Passport
- `Nodemon` is used for hot-reloading instead of `node-dev` because the files are dynamically required, so file-watching is needed to identify new files being added but not specifically required in `./models`.

## TODOs

- enable extensibility for login object? (eg. phone num, descript, other meta fields)
- sanitize all inputs in express middleware...
- support listing with filtering?
- auto-add `owner_id: String` and `{ strict: "throw", toObject: { versionKey: false } }` using Mongoose discriminators for schema inheritance?
- enable disabling all auth for all endpoints via config var
