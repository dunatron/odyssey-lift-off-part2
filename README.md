# Odyssey Lift-off II: Resolvers

# installing codeGen

`npm install -D @graphql-codegen/cli @graphql-codegen/typescript @graphql-codegen/typescript-resolvers`

```json
"scripts": {
    "compile": "tsc",
    "dev": "ts-node-dev --respawn ./src/index.ts",
    "start": "npm run compile && nodemon ./dist/index.js",
    "generate": "graphql-codegen"
  },
```

### 3. create codegen.ts

create a file called codegen.ts in the root of the server folder.

```ts
import type { CodegenConfig } from "@graphql-codegen/cli";

const config: CodegenConfig = {
  schema: "./src/schema.ts",
  generates: {
    "./src/types.ts": {
      plugins: ["typescript", "typescript-resolvers"],
    },
  },
};

export default config;
```

### 4. Add Resolvers type to resolvers

To use this type, let's open up the resolvers.ts file.

At the top, we can import the Resolvers type from the types file.

```ts
import { Resolvers } from "./types";

export const resolvers: Resolvers = {
  Query: {
    tracksForHome: (_, __, { dataSources }) => {
      return dataSources.trackAPI.getTracksForHome();
    },
  },
  Track: {
    author: ({ authorId }, _, { dataSources }) => {
      return dataSources.trackAPI.getAuthor(authorId);
    },
  },
};
```

This doesn't take care of all of our errors immediately; for one thing, we'll probably see a red squiggly appear under authorId in our Track.author resolver. Secondly, if we hover over any of our resolvers' dataSources value, we'll see that it still has an implicit type of any!

Let's take care of the dataSources issue first. If we review the Resolvers type, we'll see that it's actually a generic that accepts a type variable called ContextType.

## Resources

[CodeGen on the server for TS](https://www.apollographql.com/tutorials/lift-off-part2/08-server-codegen)
