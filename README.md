# Reproduction for issue with API routes and `basePath`

This repository is a reproduction for an issue with API routes and `basePath` with SST and open-next.

## Reproduction steps

### 1. `create-next-app` and initialize SST

```bash
npx create-next-app@latest next-sst

cd next-sst
sst init
```

### 2. Update the `next.config.mjs` file to include the `basePath`

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  basePath: '/base-path',
};

export default nextConfig;
```

### 3. Add an API route at `app/api/hello/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";

// eslint-disable-next-line @typescript-eslint/no-unused-vars
export async function GET(request: NextRequest) {
  return NextResponse.json({ message: "Hello world!" });
}
```

### 4. Use latest version of `ope-next` in `sst.config.ts`

`3.1.3` at the time of writing.

```typescript
/// <reference path="./.sst/platform/config.d.ts" />

export default $config({
  app(input) {
    return {
      name: "next-sst",
      removal: input?.stage === "production" ? "retain" : "remove",
      home: "aws",
    };
  },
  async run() {
    new sst.aws.Nextjs("MyWeb", {
      openNextVersion: "3.1.3"
    });
  },
});
```

### 5. Deploy the app

```bash
sst deploy --stage api-test
```

### 6. Open the following URLs in the browser

1. https://assignedhost.cloudfront.net/base-path
2. https://assignedhost.cloudfront.net/base-path/api/hello

Replace `assignedhost` with the actual CloudFront distribution URL.

## Expected results

Both 6.1 and 6.2 return successful responses.

## Actual results

6.1 returns a successful response, but 6.2 returns a 404 error. Note that changing the URL for 6.2 to exclude the `basePath` (https://assignedhost.cloudfront.net/api/hello) works.
