# Cloud Claw (Cloudflare + OpenClaw)

**Cloud Claw** is a containerized AI assistant solution that combines Cloudflare's powerful infrastructure with OpenClaw's intelligent capabilities.

This is a TypeScript project based on Cloudflare Workers and Cloudflare Containers. It leverages Cloudflare's infrastructure to run and manage containerized workloads.

English | [简体中文](README.zh-CN.md)

---

## Tech Stack

- **Runtime**: Cloudflare Workers + Containers
- **Language**: TypeScript (ES2024)
- **Package Manager**: pnpm
- **Container Specs**: 1 vCPU, 4GB RAM, 8GB disk
- **Core Libraries**:
  - `cloudflare:workers`: Workers standard library
  - `@cloudflare/containers`: Container management
- **Container Base**: `nikolaik/python-nodejs:python3.12-nodejs22-bookworm`
- **Storage**: TigrisFS for S3/R2 mounting

## Quick Start

[![Deploy to Cloudflare](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/miantiao-me/cloud-claw)

### Prerequisites

- Node.js (v22+)
- pnpm (v10.28.2+)
- Wrangler CLI (`npm i -g wrangler`)

### Install Dependencies

```bash
pnpm install
```

### Local Development

Start the local development server:

```bash
pnpm dev
```

### Linting

Run formatter (oxfmt) and linter (oxlint):

```bash
pnpm lint
```

### Generate Type Definitions

If you modify bindings in `wrangler.jsonc`, regenerate the type file:

```bash
pnpm cf-typegen
```

## Deployment

Deploy code to Cloudflare's global network:

```bash
pnpm deploy
```

## Project Structure

```
.
├── src/
│   ├── index.ts        # Workers entry file (ExportedHandler)
│   └── container.ts    # AgentContainer class definition (extends Container)
├── worker-configuration.d.ts # Auto-generated environment binding types
├── wrangler.jsonc      # Wrangler configuration file
├── tsconfig.json       # TypeScript configuration
└── package.json
```

## Data Persistence (S3/R2)

The container has built-in support for S3-compatible storage (such as Cloudflare R2, AWS S3). It uses `TigrisFS` to mount object storage as a local filesystem for persistent data storage.

### Environment Variables

To enable data persistence, configure the following environment variables in the container runtime environment:

| Variable                 | Description                                      | Required | Default |
| ------------------------ | ------------------------------------------------ | -------- | ------- |
| `S3_ENDPOINT`            | S3 API endpoint address                          | Yes      | -       |
| `S3_BUCKET`              | Bucket name                                      | Yes      | -       |
| `S3_ACCESS_KEY_ID`       | Access Key ID                                    | Yes      | -       |
| `S3_SECRET_ACCESS_KEY`   | Access Key Secret                                | Yes      | -       |
| `S3_REGION`              | Storage region                                   | No       | `auto`  |
| `S3_PATH_STYLE`          | Whether to use Path Style access                 | No       | `false` |
| `S3_PREFIX`              | Path prefix within the bucket (subdirectory)     | No       | (root)  |
| `TIGRISFS_ARGS`          | Additional mount arguments for TigrisFS          | No       | -       |
| `OPENCLAW_GATEWAY_TOKEN` | Gateway access token (for Web UI authentication) | Yes      | -       |

### How It Works

1. **Mount Point**: On container startup, the S3 bucket is mounted to `/data`.
2. **Workspace**: The actual workspace is located at `/data/workspace`.
3. **OpenClaw Config**: OpenClaw configuration files are stored in `/data/.openclaw` to ensure state persistence.
4. **Initialization**:
   - If the S3 bucket (or specified prefix path) is empty, the container automatically initializes the preset directory structure.
   - If S3 configuration is missing, the container falls back to non-persistent local directory mode.

### Web UI Initialization

After the first startup, OpenClaw needs to be initialized via the Web UI.
Visit the deployed URL (e.g., `https://your-worker.workers.dev`) and follow the on-screen instructions to complete setup.

## Container Lifecycle

By default, the container automatically sleeps after 10 minutes of inactivity to save resources. You can customize this behavior:

### Keep Container Always Running

To prevent the container from sleeping, modify `src/container.ts`:

```typescript
export class AgentContainer extends Container {
  sleepAfter = 'never' // Never sleep (default: '10m')
  // ...
}
```

### Activity-Based Keep-Alive (Default)

The current implementation uses smart keep-alive: the container stays active during AI conversations and sleeps during idle periods. This is achieved by calling `renewActivityTimeout()` when chat events are received:

```typescript
// In watchContainer() - resets the sleep timer on each chat completion
if (frame.event === 'chat' && frame.payload?.state === 'final') {
  this.renewActivityTimeout()
}
```

### Available Options

| `sleepAfter` Value | Behavior                                       |
| ------------------ | ---------------------------------------------- |
| `'never'`          | Container runs indefinitely                    |
| `'10m'`            | Sleep after 10 minutes of inactivity (default) |
| `'1h'`             | Sleep after 1 hour of inactivity               |
| `'30s'`            | Sleep after 30 seconds of inactivity           |

> **Note**: When sleeping, the container state is preserved. It will automatically wake up on the next request, but cold start may take a few seconds.

## Development Guidelines

For detailed development guidelines, code style, and AI agent behavior standards, please refer to [AGENTS.md](./AGENTS.md).
