# Next.js Installation and Deployment Guide

This guide will walk you through the steps to install Next.js and deploy your
Next.js application on various platforms, including Vercel, Netlify, and Docker.

## Prerequisites

Before getting started, make sure you have the following prerequisites installed
on your system:

- Node.js: [Download and install Node.js](https://nodejs.org/)
- Git: [Download and install Git](https://git-scm.com/)

## Installation

### Step 1: Install all the necessary packages

To run this Next.js app, open your terminal and run the following commands:

```bash
npm i
# Alternatively use yarn or pnpm
```

### Step 2: Start the Development Server

```bash
npm run dev
# Alternatively pnpm or yarn
```

## Deployment with Vercel

Install the Vercel CLI globally:

```bash
npm install -g vercel
```

Follow the following [vercel cli](https://www.npmjs.com/package/vercel) guide.

## Deployment with Docker

### We'll be creating our own Dockerfile soon

Create a Dockerfile in your Next.js project directory with the following
content:

```dockerfile
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
# Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Install dependencies based on the preferred package manager
COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./
RUN \
  if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
  elif [ -f package-lock.json ]; then npm ci; \
  elif [ -f pnpm-lock.yaml ]; then yarn global add pnpm && pnpm i --frozen-lockfile; \
  else echo "Lockfile not found." && exit 1; \
  fi


# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Next.js collects completely anonymous telemetry data about general usage.
# Learn more here: https://nextjs.org/telemetry
# Uncomment the following line in case you want to disable telemetry during the build.
# ENV NEXT_TELEMETRY_DISABLED 1

RUN yarn build

# If using npm comment out above and use below instead
# RUN npm run build

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
# Uncomment the following line in case you want to disable telemetry during runtime.
# ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

# Set the correct permission for prerender cache
RUN mkdir .next
RUN chown nextjs:nodejs .next

# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
# set hostname to localhost
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

**Thanks to
[Vercel](https://github.com/vercel/next.js/blob/canary/examples/with-docker/Dockerfile)
for providing us with this template**

## Conclusion

You've successfully installed Next.js and deployed your app on Vercel, Netlify,
and as a Docker container. Happy coding!
