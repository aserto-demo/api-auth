# api-auth

API Authorization using Aserto and Topaz.

This sample imports the OpenAPI definitions from three common services:
* todo list API
* rick and morty API
* petstore API

These definitions become `service` and `endpoint` objects in Topaz.

Each `endpoint` can be assigned a set of `invoker` users and groups, which entitles that user (or members of the group) to invoke the endpoint.

To make management more scalable, each service also has a `readers`, `writers`, `creators`, and `deleters` group associated with it, which are entitled in the following way:

* `<service-name>-readers`: can invoke GET endpoints
* `<service-name>-writers`: can invoke GET, PUT, PATCH endpoints
* `<service-name>-creators`: can invoke GET, PUT, PATCH, POST endpoints
* `<service-name>-deleters`: can invoke GET, PUT, PATCH, POST, DELETE endpoints

Finally, we also create 4 global groups, which have these entitlements across all services:
* `global-readers`
* `global-writers`
* `global-creators`
* `global-deleters`

## Setup

### 1. Install CLIs

Install `topaz` and `ds-load` CLIs:

```bash
brew tap aserto-dev/tap
brew install topaz
brew install ds-load
```

### 2. Create configuration

Create a new `topaz` configuration by installing the `api-auth` template:

```bash
topaz templates install api-auth https://raw.githubusercontent.com/aserto-demo/api-auth/main/templates.json
```

### 3. Import OpenAPI specs

Import the three OpenAPI specs in the `./openapi` directory as `service` and `endpoint` instances:

```bash
ds-load openapi -d ./openapi
```

This will create 4 groups per service:
* `<service-name>-readers`: can invoke GET endpoints
* `<service-name>-writers`: can invoke GET, PUT, PATCH endpoints
* `<service-name>-creators`: can invoke GET, PUT, PATCH, POST endpoints
* `<service-name>-deleters`: can invoke GET, PUT, PATCH, POST, DELETE endpoints

It will also create 4 global groups, which have these entitlements across all services:
* `global-readers`
* `global-writers`
* `global-creators`
* `global-deleters`

### 4. Entitle users to APIs

Assign users to the groups (or entitle users to be able to invoke individual endpoints).

The easiest way to do this is within the Topaz Console:

```bash
topaz console
```

You can also do this from the CLI.

For example, to make Morty be able to invoke all GET endpoints across all services:

```bash
topaz ds set relation '
{
  "relation": {
    "object_type": "group",
    "object_id": "global-readers",
    "relation": "member",
    "subject_type": "user",
    "subject_id": "morty@the-citadel.com"
  }
}'
```

### 5. Test the model

Check whether a user can invoke an endpoint.

You can do this in the Topaz "Evaluator" tab, or from the CLI.

For example, to check whether Morty can invoke the `Todo_List_API:GET:/v1/todos` endpoint:

```bash
topaz ds check '{
  "object_type": "endpoint",
  "object_id": "Todo_List_API:GET:/v1/todos",
  "relation": "can_invoke",
  "subject_type": "user",
  "subject_id": "morty@the-citadel.com"
}'
```

This should come back as `true`, since Morty is a member of the `global-readers` group.

To check whether Rick can invoke the same API, issue the following command:

```bash
topaz ds check '{
  "object_type": "endpoint",
  "object_id": "Todo_List_API:GET:/v1/todos",
  "relation": "can_invoke",
  "subject_type": "user",
  "subject_id": "rick@the-citadel.com"
}'
```

This should come back as `false`, since Rick has not been entitled to any APIs.

To allow Rick to invoke (only) this endpoint, we can make him an invoker:

```bash
topaz ds set relation '
{
  "relation": {
    "object_type": "endpoint",
    "object_id": "Todo_List_API:GET:/v1/todos",
    "relation": "invoker",
    "subject_type": "user",
    "subject_id": "rick@the-citadel.com"
  }
}'
```

Now Rick should be able to invoke the API:

```bash
topaz ds check '{
  "object_type": "endpoint",
  "object_id": "Todo_List_API:GET:/v1/todos",
  "relation": "can_invoke",
  "subject_type": "user",
  "subject_id": "rick@the-citadel.com"
}'
```

This should evaluate as `true`.
