# ----------------------------
# Stage 1: Build React App
# ----------------------------
FROM node:18-alpine AS build

# Set working directory
WORKDIR /app

# Copy dependency files first (better cache)
COPY package.json package-lock.json* ./

# Install dependencies
# Use npm ci if package-lock.json exists, otherwise fallback to npm install
RUN if [ -f package-lock.json ]; then \
      npm ci; \
    else \
      npm install; \
    fi

# Copy the rest of the source code
COPY . .

# Backend IP configuration
ARG BACKEND_HOST
ARG BACKEND_PORT=8080

RUN apk add --no-cache curl && \
    if [ -n "$BACKEND_HOST" ]; then \
        BACKEND_IP="$BACKEND_HOST"; \
    else \
        BACKEND_IP=$(curl -s ifconfig.me 2>/dev/null || echo "localhost"); \
    fi && \
    echo "Detected backend IP: $BACKEND_IP" && \
    if [ -f .env ]; then \
        sed -i "s/\$BACKEND/$BACKEND_IP/g" .env; \
        echo "Updated .env with backend IP: $BACKEND_IP"; \
    else \
        echo "VITE_API_URL=http://$BACKEND_IP:$BACKEND_PORT/api" > .env; \
        echo "VITE_API_BASE_URL=http://$BACKEND_IP:$BACKEND_PORT" >> .env; \
        echo "VITE_APP_TITLE=EasyCRUD Student Registration" >> .env; \
        echo "Created new .env file with backend IP: $BACKEND_IP"; \
    fi

# Build production-ready React app
RUN npm run build

# ----------------------------
# Stage 2: Nginx Server
# ----------------------------
FROM nginx:alpine

# Install curl for health checks
RUN apk add --no-cache curl

# Copy built app to nginx html folder
COPY --from=build /app/dist /usr/share/nginx/html

# Copy custom nginx config
COPY nginx-simple.conf /etc/nginx/conf.d/default.conf

# Ensure correct ownership
RUN chown -R nginx:nginx /usr/share/nginx/html /var/cache/nginx /var/log/nginx /etc/nginx/conf.d

# Expose nginx default port
EXPOSE 80

# Healthcheck
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
  CMD curl -f http://localhost || exit 1

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
