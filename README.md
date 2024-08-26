# Cloudflare Worker Template OpenAPI Service

This project demonstrates a Cloudflare Worker implementation using OpenAPI 3.1 with [@hono/zod-openapi](https://github.com/honojs/hono). It serves as a template for building OpenAPI-compliant Workers that automatically generate the `openapi.json` schema from code and validate incoming requests against defined parameters and request bodies.

## Features

- OpenAPI 3.1 compliant
- Automatic `openapi.json` schema generation
- Request validation
- Swagger UI for easy API exploration
- Support for R2, KV, and Durable Objects
- Structured project layout for scalability

## Getting Started

1. Sign up for [Cloudflare Workers](https://workers.dev) (free tier available)
2. Clone this repository
3. Install dependencies:
   ```
   npm install
   ```
4. Log in to your Cloudflare account:
   ```
   wrangler login
   ```
5. Deploy the API:
   ```
   wrangler deploy
   ```

## Project Structure

- `src/index.ts`: Main application entry point
- `src/endpoints/`: Individual endpoint definitions
- `src/pkg/`: Shared utilities and configurations
- `test/`: Test files

## Development

1. Start the local development server:
   ```
   wrangler dev
   ```
2. Open `http://localhost:9000/` in your browser to access the Swagger UI
3. Make changes in the `src/` directory; the server will automatically reload

## Adding New Endpoints

1. Create a new file in `src/endpoints/`
2. Define your route using `createRoute` from `@hono/zod-openapi`
3. Implement the handler function
4. Register the route in `src/index.ts`

Example:

```typescript:src/endpoints/r2/routes.ts
import { createRoute, z } from "@hono/zod-openapi";
import { openApiErrorResponses } from "../../pkg/errors";
import { RecordSchema } from "./types";

export const getRoute = createRoute({
  tags: ["r2"],
  operationId: "v1.r2.get",
  method: "get",
  path: "/v1/r2/{key}",
  request: {
    params: z.object({
      key: z.string(),
    }),
  },
  responses: {
    200: {
      description: "The requested record",
      content: {
        "application/json": {
          schema: RecordSchema,
        },
      },
    },
    ...openApiErrorResponses,
  },
});

export const putRoute = createRoute({
  tags: ["r2"],
  operationId: "v1.r2.put",
  method: "put",
  path: "/v1/r2/{key}",
  request: {
    params: z.object({
      key: z.string(),
    }),
    body: {
      content: {
        "application/json": {
          schema: RecordSchema,
        },
      },
    },
  },
  responses: {
    201: {
      description: "Record stored successfully",
      content: {
        "application/json": {
          schema: z.object({
            message: z.string(),
            id: z.string().uuid(),
          }),
        },
      },
    },
    ...openApiErrorResponses,
  },
});

export const deleteRoute = createRoute({
  tags: ["r2"],
  operationId: "v1.r2.delete",
  method: "delete",
  path: "/v1/r2/{key}",
  request: {
    params: z.object({
      key: z.string(),
    }),
  },
  responses: {
    200: {
      description: "Record deleted successfully",
      content: {
        "application/json": {
          schema: z.object({
            message: z.string(),
          }),
        },
      },
    },
    ...openApiErrorResponses,
  },
});
```

## Documentation

For more detailed information on using `@hono/zod-openapi`, refer to the [official Hono documentation](https://hono.dev/guides/zod-openapi).

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is open source and available under the [MIT License](LICENSE).

## OpenAPI Definition Generation

This project includes a script to automatically generate the OpenAPI definition from the code. Here's how it works:

1. The OpenAPI definition is generated using a custom script located at `scripts/openapi.ts`.

2. To run the script, use the following command:
   ```
   bun run openapi
   ```

3. This script does the following:
   - Imports the main application from `src/index.ts`
   - Generates the OpenAPI document using Hono's built-in functionality
   - Writes the resulting schema to `api.json` in the project root

4. The generation script is automatically run as part of the pre-commit hook to ensure the API definition is always up-to-date.

5. You can find the script configuration in `package.json`:
   ```json
   "scripts": {
     "openapi": "bun scripts/openapi.ts",
     "precommit": "npm run openapi && npm run format"
   }
   ```

This approach ensures that your OpenAPI definition always reflects the current state of your API implementation.
