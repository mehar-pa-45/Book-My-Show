# Use Node LTS image
FROM node:18-alpine

# Create app directory
WORKDIR /app

# Copy dependency files first (Docker cache optimization)
COPY package*.json ./

# Install dependencies
RUN npm install --legacy-peer-deps

# Fix file watching issue inside Docker (Chokidar)
ENV CHOKIDAR_USEPOLLING=true

# Fix OpenSSL issue
ENV NODE_OPTIONS=--openssl-legacy-provider

# Copy project files
COPY . .

# Expose application port
EXPOSE 3000

# Start application
CMD ["npm", "start"]
