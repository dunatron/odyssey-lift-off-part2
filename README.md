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

### 5 create models.ts

create a models.ts file in the root directory

```ts
export type TrackModel = {
  id: string;
  title: string;
  authorId: string;
  thumbnail: string;
  length: number;
  modulesCount: number;
};

export type AuthorModel = {
  id: string;
  name: string;
  photo: string;
};
```

updated our codegen file

```ts
import type { CodegenConfig } from "@graphql-codegen/cli";

const config: CodegenConfig = {
  schema: "./src/schema.ts",
  generates: {
    "./src/types.ts": {
      plugins: ["typescript", "typescript-resolvers"],
      config: {
        contextType: "./context#DataSourceContext",
        mappers: {
          Track: "./models#TrackModel",
          Author: "./models#AuthorModel",
        },
      },
    },
  },
};

export default config;
```

run codegen again.

This has an added benefit in that we can make our dataSources have types

```ts
import { RESTDataSource } from "@apollo/datasource-rest";
import { TrackModel, AuthorModel } from "../models";

export class TrackAPI extends RESTDataSource {
  baseURL = "https://odyssey-lift-off-rest-api.herokuapp.com/";

  getTracksForHome() {
    return this.get<TrackModel[]>("tracks");
  }

  getAuthor(authorId: string) {
    return this.get<AuthorModel>(`author/${authorId}`);
  }
}
```

## Full Guide

### Resolver types

We've replaced mock data with real data by hooking up resolvers and a datasource. But currently, our `resolvers.ts` file isn't taking advantage of the benefits of TypeScript.

If we hover over our `resolvers` object, we'll see that TypeScript doesn't have a clear idea of what kinds of data each resolver function accepts and returns! Instead, we see that almost everything is inferred as an `any` type.

We can use the types and fields in our schema to help clarify these data types - but we don't have to type them out manually! We can make use of a GraphQL Code Generator on the backend as well.

### GraphQL Codegen

Let's start by navigating to a terminal in the `server` directory, and installing three packages: `@graphql-codegen/cli, @graphql-codegen/typescript and @graphql-codegen/typescript-resolvers.`

`npm install -D @graphql-codegen/cli @graphql-codegen/typescript @graphql-codegen/typescript-resolvers`

Next, let's create a codegen command that we can run from our `server/package.json` file. Add a new entry called `generate` under the `scripts` object, and set it to `graphql-codegen`.

```json
"scripts": {
    "compile": "tsc",
    "dev": "ts-node-dev --respawn ./src/index.ts",
    "start": "npm run compile && nodemon ./dist/index.js",
    "generate": "graphql-codegen"
  },
```

To run this command successfully, we need a file that will contain the instructions the GraphQL Code Generator can follow.

Let's create a file called `codegen.ts` in the root of the `server` folder. We'll use the same boilerplate we started with on the frontend.

codegen.ts

```ts
import type { CodegenConfig } from "@graphql-codegen/cli";

const config: CodegenConfig = {};

export default config;
```

The first property we can set in the config object is where our schema lives. We'll pass in the file path relative to this current directory.

codegen.ts

```ts
import type { CodegenConfig } from "@graphql-codegen/cli";

const config: CodegenConfig = {
  schema: "./src/schema.ts",
};

export default config;
```

Next, we define where the generated types should be outputted. Under a key called `generates` we'll add an object containing our desired path of `src/types.ts`. This will create a new file called `types.ts` in our server's `src` folder.

```ts
import type { CodegenConfig } from "@graphql-codegen/cli";

const config: CodegenConfig = {
  schema: "./src/schema.ts",
  generates: {
    "./src/types.ts": {},
  },
};

export default config;
```

Note: The path to the output file is contained in quotes, and it ends with a colon (:) - this is because we're about to specify some additional properties beneath it!

Next, let's tell the Code Generator which `plugins` to use, under a plugins key. This is an array that contains `typescript` and t`ypescript-resolvers`, to point to our two plugins.

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

So, what are these plugins responsible for in the code generation process?

Well, `@graphql-codegen/typescript` is the base plugin needed to generate TypeScript types from our schema.

And `@graphql-codegen/typescript-resolvers` does something similar - it will review our schema, consider the types and fields we've defined, and output the types we need to accurately describe what data our resolver functions use and return.

With that, we're ready to generate some types!

### Generating types

We have everything we need to run the codegen command. Open up a terminal in the server directory and run the following command.

`npm run generate`

After a few moments, we should see that a new file has been added to our server's src directory - `types.ts`! Let's take a look.

This file contains many of the same definitions that we generated for use on the frontend, but with some special additions. We can see that we have a number of new types that are specific to resolver functions.

At the bottom of the file, we'll find `type Resolvers`. This is the type that we'll use to more accurately describe the data our resolver functions are capable of returning, based on the schema fields they map to.

```ts
export type Resolvers<ContextType = any> = {
  Author?: AuthorResolvers<ContextType>;
  Query?: QueryResolvers<ContextType>;
  Track?: TrackResolvers<ContextType>;
};
```

To use this type, let's open up the `resolvers.ts` file.

At the top, we can import the `Resolvers` type from the `types` file.

```ts
import { Resolvers } from "./types";
```

Next, let's update the type for the `resolvers` object, and explicitly tell TypeScript that it is of type `Resolvers`.

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

This doesn't take care of all of our errors immediately; for one thing, we'll probably see a red squiggly appear under `authorId` in our `Track.author` resolver. Secondly, if we hover over any of our resolvers' `dataSources` value, we'll see that it still has an implicit type of `any`!

Let's take care of the `dataSources` issue first. If we review the Resolvers type, we'll see that it's actually a generic that accepts a type variable called ContextType.

```ts
export type Resolvers<ContextType = any> = {
  Author?: AuthorResolvers<ContextType>;
  Query?: QueryResolvers<ContextType>;
  Track?: TrackResolvers<ContextType>;
};
```

The `ContextType` variable here can be used to represent the type of whatever we set on our server's `contextValue` parameter. It lets us describe more accurately what kind of data our resolver functions have access to, what they might be reaching out for, what methods are available for them to call, and what kinds of data those methods will return.

Right now, our resolvers destructure `contextValue` for the `dataSources` property, but we can't automatically infer what types of data or methods are available within `dataSources` - that's because though the Code Generator knows about our types and fields, it has no clue how we're resolving the data with the `TrackAPI` class! By including this piece of the puzzle, we can round out our `Resolvers` type definition and empower TypeScript to help us out as we code.

Let's start by defining the value that we'll pass in to the Resolvers type as our `ContextType` variable.

In the `server/src` folder, we'll create a new file called `context.ts`. This is where we'll define the type that describes the context we pass to our server.

context.ts;

```ts
// TODO
```

Let's define and export a type called `DataSourceContext`. Inside of `DataSourceContext` we'll define a property called `dataSources`, which is an object.

context.ts

```ts
export type DataSourceContext = {
  dataSources: {};
};
```

Let's jump back over to our `index.ts` file. When we set up our `TrackAPI` datasource class, we defined a property called `trackAPI` and instantiated the `TrackAPI` class.

index.ts

```ts
return {
  dataSources: {
    trackAPI: new TrackAPI({ cache }),
  },
};
```

We can give our `DataSourceContext` type the same `trackAPI` property, but as its value we'll set `TrackAPI`. We don't need to instantiate the class - this is enough to give our type definition the information it needs about the methods and properties available on the datasource class.

context.ts

```ts
export type DataSourceContext = {
  dataSources: {
    trackAPI: TrackAPI;
  };
};
```

Finally, let's import the `TrackAPI` class from the `datasources/track-api.ts` file.

context.ts

```ts
import { TrackAPI } from "./datasources/track-api";

export type DataSourceContext = {
  dataSources: {
    trackAPI: TrackAPI;
  };
};
```

### Updating the codegen config

With our `DataSourceContext` type defined, we can update our `codegen.ts` file to take it into consideration.

Just below the `plugins` key, we can add a new `config` property. This is an object that specifies a `contextType`.

```ts
const config: CodegenConfig = {
  schema: "./src/schema.ts",
  generates: {
    "./src/types.ts": {
      plugins: ["typescript", "typescript-resolvers"],
      config: {
        contextType:
      },
    },
  },
};
```

As the value of `contextType`, we'll pass the filepath to our `context.ts` file, relative to the `./src/types.ts` file. Our `context.ts` file is located in the same `src` folder, so our path is `"./context"`. Finally, to point to the type we defined in the file, we can tack on `#DataSourceContext` to the end of the file path.

```ts
const config: CodegenConfig = {
  schema: "./src/schema.ts",
  generates: {
    "./src/types.ts": {
      plugins: ["typescript", "typescript-resolvers"],
      config: {
        contextType: "./context#DataSourceContext",
      },
    },
  },
};
```

Note: Make sure that there is NOT a forward slash (/) separating `./context` and `#DataSourceContext` in the file path you specify.

Finally, let's run our codegen command again in the server directory!

`npm run generate`

Now when we reopen the `types.ts` file and scroll down to our `Resolvers` type, we'll see that the `ContextType` defaults to `DataSourceContext` unless we specify otherwise!

```ts
export type Resolvers<ContextType = DataSourceContext> = {
  Author?: AuthorResolvers<ContextType>;
  Query?: QueryResolvers<ContextType>;
  Track?: TrackResolvers<ContextType>;
};
```

And back in `resolvers.ts`, we can hover over `dataSources` to see that our type has been inferred correctly. (And if we try to use a method that our resolvers don't have access to, we'll get an error telling us!)

Next, let's wrap up our code generation for the server by tackling the red squiggly under `authorId` in the `Track.author` resolver.

### Adding a Model type

First, let's talk about what's going wrong with the authorId. If you hover over it, you'll see the following error.

`Property authorId does not exist on type Track.`

And if we reference the `Track` type in our `schema.ts` file... we'll see that this error is, in fact, correct! Our `Track` type doesn't actually have an `authorId` field. Instead, it has an `author` field.

```gql
"A track is a group of Modules that teaches about a specific topic"
type Track {
  id: ID!
  "The track's title"
  title: String!
  "The track's main author"
  author: Author!
  "The track's main illustration to display in track card or track page detail"
  thumbnail: String
  "The track's approximate length to complete, in minutes"
  length: Int
  "The number of modules this track contains"
  modulesCount: Int
}
```

So... where's the disconnect?

Well, we saw in the lesson [Exploring our data](https://www.apollographql.com/tutorials/lift-off-part2/02-exploring-our-data) that each track object we query contains a bunch of data - including an `authorId` field!

```json
{
    "id": "c_0",
    "thumbnail": "https://res.cloudinary.com/dety84pbu/image/upload/v1598465568/nebula_cat_djkt9r.jpg",
    "topic": "Cat-stronomy",
    "authorId": "cat-1",
    "title": "Cat-stronomy, an introduction",
    "description": "Curious to learn what Cat-stronomy is all about? Explore the planetary and celestial alignments and how they have affected our space missions.",
    "numberOfViews": 0,
    "createdAt": "2018-09-10T07:13:53.020Z",
    "length": 2377,
    "modulesCount": 10,
    "modules": ["l_0", "l_1", "l_2", "l_3", "l_4", "l_5", "l_6", "l_7", "l_8", "l_9"]
  },
```

And in the lesson for [Implementing query resolvers](https://www.apollographql.com/tutorials/lift-off-part2/06-implementing-query-resolvers), we saw how we could pluck the authorId field from each track object to call our datasource for information about a particular author.

resolvers.ts

```ts
Track: {
    author: ({ authorId }, _, { dataSources }) => {
      return dataSources.trackAPI.getAuthor(authorId);
    },
  },
```

But in our schema, we represented this connection between a `Track` and its `Author` more dynamically. By giving `Track` an `author` field of type `Author`, we could utilize the flexibility of GraphQL to easily traverse from one type to another - with our resolver functions doing all the magic to connect data behind the scenes!

We would be a lot more limited if we could only query for a `Track`'s `authorId`; we'd have to make another GraphQL query, instead of getting data for tracks, and their authors, all at once!

In this case, our generated types are trying to help us out. They know what we've defined in the schema, and they warn us when we try to access anything outside of those definitions! By using a Model type, we can overcome some of this mismatch between how our backend datasources are implemented, and how we work with them in our GraphQL API.

To see this illustrated, let's create a new file in our `server/src` folder called `models.ts`. Right away, we can export a new type called `TrackModel`.

server/src/models.ts

```ts
export type TrackModel = {};
```

Next, we just need to give the Code Generator the information it's missing: namely, what our backend implementation of a track object actually looks like. This means we'll include all of the data fields we might want to access and use. We can use our `Track` type as a reference for the fields we need to include.

```gql
"A track is a group of Modules that teaches about a specific topic"
type Track {
  id: ID!
  "The track's title"
  title: String!
  "The track's main author"
  author: Author!
  "The track's main illustration to display in track card or track page detail"
  thumbnail: String
  "The track's approximate length to complete, in minutes"
  length: Int
  "The number of modules this track contains"
  modulesCount: Int
}
```

For the most part, the fields on `Track` map one-to-one to the actual fields on the underlying data object. We'll include all of these fields on our `TrackModel` type, with the exception of `author`. As we saw, `author` does not exist in our database. Instead, we use the `authorId` field to map a `Track` object to a particular `Author` object. For this reason, we'll include `authorId` as a `string` type on `TrackModel` instead.

server/src/models.ts

```ts
export type TrackModel = {
  id: string;
  title: string;
  authorId: string;
  thumbnail: string;
  length: number;
  modulesCount: number;
};
```

Next, we can incorporate our new `TrackModel` information into our `codegen.ts` file.

Inside of the `config` object, we'll add a new key called `mappers`. `mappers` is an object which we'll have a `Track` key. As the value of `Track`, we'll pass the path `./models#TrackModel` to reference the `TrackModel` type directly.

codegen.ts

```ts
const config: CodegenConfig = {
  schema: "./src/schema.ts",
  generates: {
    "./src/types.ts": {
      plugins: ["typescript", "typescript-resolvers"],
      config: {
        contextType: "./context#DataSourceContext",
        mappers: {
          Track: "./models#TrackModel",
        },
      },
    },
  },
};
```

Fantastic! Let's run our codegen command in the server directory one last time to update our types.

`npm run generate`

After our types are regenerated, we should see in our `resolvers.ts` file that the red squiggly line under `authorId` in the `Track.author` resolver function has vanished! The type of the `parent` argument being passed to this function has updated to be `TrackModel`. Our codegen is complete!

### Updating type definitions

Adding our `TrackModel` type has another benefit: we can update our `TrackAPI` class methods to more accurately define what kind of data they return.

Open up `src/datasources/track-api.ts` in your `server` directory. We'll start by importing the `TrackModel` type from our `models` file.

```ts
import { TrackModel } from "../models";
```

Next, we'll use this `TrackModel` to annotate the `this.get` method called in `getTracksForHome`. This method accepts a generic, where we can indicate that it returns an array of `TrackModel` objects.

src/datasources/track-api.ts

```ts
getTracksForHome() {
  return this.get<TrackModel[]>("tracks");
}
```

We can do the same thing for our `getAuthor` method, but we'll first need to create an `AuthorModel`. Inside of our `models.ts` file, let's add a new type.

models.ts

```ts
export type AuthorModel = {};
```

In our schema, our `Author` type only uses three fields that are returned from the REST data source: `id`, `name`, and `photo`. Let's go ahead and include them, annotating them with the proper TypeScript types.

models.ts

```ts
export type TrackModel = {
  id: string;
  title: string;
  authorId: string;
  thumbnail: string;
  length: number;
  modulesCount: number;
};

export type AuthorModel = {
  id: string;
  name: string;
  photo: string;
};
```

Let's also update our `mappers` property in the `codegen.ts` file as well so that it includes `AuthorModel`.

codegen.ts

```ts
// Other codegen configuration
mappers: {
  Track: "./models#TrackModel",
  Author: "./models#AuthorModel"
},
```

Next, we can import our `AuthorModel` into the `track-api.ts` file alongside `TrackModel`. The `getAuthor` method resolves to a single `Author` object, so we can update the annotation.

src/datasources/track-api.ts

```ts
import { RESTDataSource } from "@apollo/datasource-rest";
import { TrackModel, AuthorModel } from "../models";

export class TrackAPI extends RESTDataSource {
  baseURL = "https://odyssey-lift-off-rest-api.herokuapp.com/";

  getTracksForHome() {
    return this.get<TrackModel[]>("tracks");
  }

  getAuthor(authorId: string) {
    return this.get<AuthorModel>(`author/${authorId}`);
  }
}
```

With that, our data sources are now fully type-annotated!

## Resources

[CodeGen on the server for TS](https://www.apollographql.com/tutorials/lift-off-part2/08-server-codegen)
