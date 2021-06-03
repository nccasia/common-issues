docs: https://www.apollographql.com/docs/apollo-server/performance/caching/
issue: @cacheControl does not work unless defaultMaxAge is set (read more https://spectrum.chat/apollo/apollo-server/cachecontrol-does-not-work-unless-defaultmaxage-is-set~0e4dc3c5-7cdc-40be-b833-0b2d436d1288)
solution: add plugin "apollo-cache-control" to retrive @cacheControl config from schema (readmore https://github.com/apollographql/apollo-cache-control)

Sample:
```js
import { ApolloServer } from "apollo-server-lambda";

import context from "./context";
import { BanksyApi } from "./datasources";
import schema from "./schema";
import { ApolloServerPlugin } from "apollo-server-plugin-base";
import responseCachePlugin from 'apollo-server-plugin-response-cache';
import { plugin as cacheControlPlugin } from 'apollo-cache-control'

export const server = new ApolloServer({
  schema,
  context,
  cache: /* Cache manager implementation */,
  persistedQueries: {
    cache: redis,
    ttl: 86400
  },
  cacheControl: true,
  plugins: [
    cacheControlPlugin(),
    responseCachePlugin() as ApolloServerPlugin<Record<string, string>>,
  ]
  tracing: false,
  debug: false
});

```
