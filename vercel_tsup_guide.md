
# Modern Node.js & TypeScript Vercel Deployment Guide

A comprehensive, production-ready guide to configuring, bundling, and deploying high-performance Node.js and TypeScript services on Vercel using `tsup`.

---

## 1. Overview & Architecture

Deploying TypeScript applications to serverless environments like Vercel requires an optimized build pipeline. Standard compilation with `tsc` often leaves multiple output files and raw `import`/`require` statements that may cause runtime module resolution errors. 

By leveraging **tsup** (powered by `esbuild`), we bundle the entire application into a single, self-contained JavaScript bundle (`dist/server.js`) with shims for Node.js built-ins and CommonJS interoperability.

### key Benefits
- **Lightning Fast Builds**: Powered by `esbuild` under the hood.
- **Single File Output**: Avoids complex `node_modules` path resolution in serverless runtimes.
- **CJS/ESM Interop**: Automatic polyfilling for `require()` inside ESM bundles.

---

## 2. Package Configuration (`package.json`)

Configure your `package.json` scripts to support local development (`tsx`) and zero-config deployment builds (`tsup`).

```json
{
  "name": "vercel-tsup-service",
  "version": "1.0.0",
  "scripts": {
    "start": "node dist/server.js",
    "dev": "tsx watch ./src/server.ts",
    "build": "tsup"
  },
  "dependencies": {
    "express": "^4.19.2"
  },
  "devDependencies": {
    "@types/express": "^4.17.21",
    "@types/node": "^20.11.0",
    "tsup": "^8.0.2",
    "tsx": "^4.7.1",
    "typescript": "^5.3.3"
  }
}
```

---

## 3. Bundler Setup (`tsup.config.ts`)

The `tsup.config.ts` file handles entry-point resolution, output directory cleanup, sourcemap generation, and module shimming.

```typescript
import { defineConfig } from "tsup";

export default defineConfig({
  entry: ["src/server.ts"],
  format: ["esm", "cjs"],
  target: "esnext",
  outDir: "dist",
  clean: true,
  bundle: true,
  splitting: false,
  sourcemap: true,
  // Add banner to shim require() for legacy CJS dependencies in ESM mode
  banner: {
    js: `
      import { createRequire } from 'module';
      const require = createRequire(import.meta.url);
    `,
  },
});
```

---

## 4. TypeScript Compiler Settings (`tsconfig.json`)

Ensure `tsconfig.json` properly scoped source inclusions and module resolutions matching your modern runtime.

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

---

## 5. Vercel Serverless Routing (`vercel.json`)

Define how Vercel routes incoming traffic to your compiled serverless handler in `dist/server.js`.

```json
{
  "version": 2,
  "builds": [
    {
      "src": "dist/server.js",
      "use": "@vercel/node"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "dist/server.js"
    }
  ]
}
```

---

## 6. CLI Deployment Sequence

Execute these commands from your terminal to install the CLI, authenticate, and push your deployment live to Vercel:

```bash
# 1. Install Vercel CLI globally
npm i -g vercel

# 2. Login to your Vercel account
vercel login

# 3. Deploy directly to Production
vercel --prod
```

> **Pro Tip:** Set up Vercel GitHub integration for automated Continuous Integration (CI) on every push to `main`!
