# Multi-stage build for production
FROM node:22-alpine AS base
ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN corepack enable

FROM base AS build
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --frozen-lockfile
COPY . .
ENV NODE_ENV=production
RUN pnpm run build

FROM base AS production
WORKDIR /app
ENV NODE_ENV=production

# Install serve to run the production build
RUN pnpm add -g serve

# Copy only the built files
COPY --from=build /app/dist ./dist

EXPOSE 3000

# Serve the production build
CMD ["serve", "-s", "dist", "-l", "3000"]