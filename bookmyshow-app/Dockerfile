############################
# BUILD STAGE
############################
FROM node:18 AS builder

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

# ✅ FIX OPENSSL ERROR
ENV NODE_OPTIONS=--openssl-legacy-provider

RUN npm run build

############################
# PRODUCTION STAGE
############################
FROM nginx:alpine

RUN rm -rf /usr/share/nginx/html/*

COPY --from=builder /app/build /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
