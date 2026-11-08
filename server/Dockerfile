FROM node:22-alpine AS builder

WORKDIR /app

# Temporary URL for Prisma config during image build
ENV DATABASE_URL="postgresql://build:build@localhost:5432/build"

# Copy package files
COPY package*.json ./

# Copy Prisma schema and configuration before npm ci
COPY prisma ./prisma
COPY prisma.config.ts ./

# Configure npm and install dependencies
RUN npm config set fetch-timeout 600000 \
    && npm config set fetch-retries 5 \
    && npm config set maxsockets 3 \
    && npm ci --no-audit --no-fund

# Copy application source
COPY . .

# Compile TypeScript
RUN npm run build


FROM node:22-alpine AS production

WORKDIR /app

ENV NODE_ENV=production

COPY package*.json ./

# Install production dependencies only
RUN npm ci --omit=dev --ignore-scripts --no-audit --no-fund

# Copy compiled application, including generated Prisma Client
COPY --from=builder /app/dist ./dist

EXPOSE 3000

CMD ["node", "dist/server.js"]
