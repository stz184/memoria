#syntax=docker/dockerfile:1.4
# Base stage for dev and build
FROM node:23-alpine as builder_base

# Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
RUN apk add --no-cache libc6-compat

WORKDIR /srv/app

RUN corepack enable && \
	corepack prepare --activate pnpm@latest && \
	pnpm config -g set store-dir /.pnpm-store


# Deps stage, preserve dependencies in cache as long as the lockfile isn't changed
FROM builder_base AS deps

COPY --link .npmrc pnpm-lock.yaml*  ./
RUN pnpm fetch

COPY --link . .
RUN pnpm install --prefer-offline


# Development image
FROM deps as dev

EXPOSE 4101
ENV PORT 4101

CMD ["sh", "-c", "pnpm install --prefer-offline; pnpm run dev"]


FROM builder_base AS builder
COPY --link . .
COPY --from=deps --link /srv/app/node_modules ./node_modules

RUN pnpm run build


# Production image, copy all the files and run next
FROM node:18-alpine AS prod
WORKDIR /srv/app

ENV NODE_ENV production

COPY --from=builder --link /srv/app/ .

CMD ["node", " dist/src/index.js"]