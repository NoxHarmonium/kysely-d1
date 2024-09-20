# @noxharmonium/kysely-d1

_Temporary_ fork of https://github.com/aidenwallis/kysely-d1 that is published on NPM under "@noxharmonium/kysely-d1"
until the changes in https://github.com/aidenwallis/kysely-d1/pull/25 are published.

I really needed the ESM changes to get vitest to work so I published this to make my life easier.
I'll delete it as soon as that PR is merged so please don't depend on this. I probably won't pull in any other updates.

```bash
npm i @noxharmonium/kysely-d1
```

This project was largely adapted from [kysely-planetscale](https://github.com/depot/kysely-planetscale).

## Usage

Pass your D1 binding into the dialect in order to configure the Kysely client. Follow [these docs](https://developers.cloudflare.com/d1/get-started/#4-bind-your-worker-to-your-d1-database) for instructions on how to do so.

```typescript
import { Kysely } from 'kysely';
import { D1Dialect } from '@noxharmonium/kysely-d1';

export interface Env {
  DB: D1Database;
}

interface KvTable {
  key: string;
  value: string;
}

interface Database {
  kv: KvTable;
}

export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const { searchParams } = new URL(request.url);
    const key = searchParams.get('key');
    if (!key) {
      return new Response('No key defined.', { status: 400 });
    }

    // Create Kysely instance with kysely-d1
    const db = new Kysely<Database>({ dialect: new D1Dialect({ database: env.DB }) });

    // Read row from D1 table
    const result = await db.selectFrom('kv').selectAll().where('key', '=', key).executeTakeFirst();
    if (!result) {
      return new Response('No value found', { status: 404 });
    }

    return new Response(result.value);
  },
};
```

There is a working [example](example) also included, which implements a K/V style store using D1.
