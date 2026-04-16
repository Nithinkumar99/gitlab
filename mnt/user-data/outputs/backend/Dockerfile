# ── Stage 1: Install dependencies ─────────────────────────────
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev

# ── Stage 2: Production image ──────────────────────────────────
FROM node:20-alpine
LABEL maintainer="your-team@example.com"

# Add non-root user for security
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Drop to non-root
USER appuser

EXPOSE 3001
HEALTHCHECK --interval=15s --timeout=5s \
  CMD wget -qO- http://localhost:3001/health || exit 1

CMD ["node", "server.js"]
