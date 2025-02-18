# Étape de base
FROM node:18-alpine AS base
WORKDIR /app

# Étape pour installer les dépendances
FROM base AS deps
RUN apk add --no-cache libc6-compat
COPY package.json pnpm-lock.yaml* ./
RUN corepack enable && corepack prepare pnpm@8.6.1 --activate && pnpm install --frozen-lockfile

# Étape de build
FROM node:18-alpine AS builder
WORKDIR /app
# Active Corepack et prépare explicitement pnpm dans ce stage
RUN corepack enable && corepack prepare pnpm@8.6.1 --activate
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN pnpm prisma:generate
# Augmente la mémoire disponible pour la build Next.js
RUN NODE_OPTIONS="--max-old-space-size=8192" pnpm run build

# Image finale de production
FROM node:18-alpine AS runner
WORKDIR /app
ENV NODE_ENV production
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nextjs
# Copier les fichiers optimisés du build
COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs
EXPOSE 3000
ENV PORT 3000
CMD ["node", "server.js"]
