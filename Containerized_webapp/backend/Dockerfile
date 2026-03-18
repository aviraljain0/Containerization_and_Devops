# Stage 1: Builder
FROM node:18-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm install --production

# Stage 2: Runtime
FROM node:18-alpine
WORKDIR /app

# Copy dependencies from builder
COPY --from=builder /app/node_modules ./node_modules
COPY . .

# Use non-root user (node is built into the node alpine image)
USER node

# Expose port
EXPOSE 3000

# Start command
CMD ["npm", "start"]