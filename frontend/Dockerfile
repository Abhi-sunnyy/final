# Frontend Dockerfile
FROM node:18-alpine AS deps

WORKDIR /app

# Install dependencies only when needed
COPY package.json package-lock.json ./
RUN npm ci

# Rebuild the source code only when needed
FROM node:18-alpine AS builder

WORKDIR /app

# Copy dependencies from deps stage
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Set environment variables for build
ENV NEXT_TELEMETRY_DISABLED 1
ENV NODE_ENV production

# Build the application
RUN npm run build

# Production image, copy all the files and run next
FROM node:18-alpine AS runner

WORKDIR /app

# Set environment variables
ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1
ENV PORT 3000
ENV NEXT_PUBLIC_API_URL http://localhost:5000
ENV NEXTAUTH_URL http://localhost:3000
ENV NEXTAUTH_SECRET group21

# Create a non-root user
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# Copy necessary files
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

# Set correct permissions
RUN chown -R nextjs:nodejs /app

# Switch to non-root user
USER nextjs

# Expose port
EXPOSE ${PORT}

# Health check
HEALTHCHECK --interval=60s --timeout=60s --start-period=5s --retries=5 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:${PORT} || exit 1

# Start the application
CMD ["npm", "start"] 