#syntax=docker/dockerfile:1.4
# Base stage for dev and build
FROM node:23-alpine as builder_base

# Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
RUN apk add --no-cache libc6-compat

WORKDIR /srv/app

RUN corepack enable && \
	corepack prepare --activate pnpm@latest && \
	pnpm config -g set store-dir /.pnpm-store

# Next.js collects completely anonymous telemetry data about general usage.
# Learn more here: https://nextjs.org/telemetry
# Uncomment the following line in case you want to disable telemetry during dev and build.
# ENV NEXT_TELEMETRY_DISABLED 1


# Deps stage, preserve dependencies in cache as long as the lockfile isn't changed
FROM builder_base AS deps

COPY --link package.json pnpm-lock.yaml* ./
RUN pnpm fetch

COPY --link . .

RUN if [ -f pnpm-lock.yaml ]; then \
	pnpm install -r --offline; \
else \
	pnpm install -r; \
fi


# Development image
FROM deps as dev

EXPOSE 3000
ENV PORT 3000

CMD ["sh", "-c", "pnpm install -r --offline; pnpm run dev"]


FROM builder_base AS builder
COPY --link . .
COPY --from=deps --link /srv/app/node_modules ./node_modules

RUN pnpm run build


# Production image, copy all the files
FROM node:23-alpine AS prod
WORKDIR /srv/app

ENV NODE_ENV production
ENV PORT 3101

COPY --from=builder /srv/app/build ./build
CMD [ "npx", "serve", "-l", "$PORT", "build" ]