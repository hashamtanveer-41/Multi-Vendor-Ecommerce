# Stage 1: Build React App
FROM node:18-bullseye-slim AS builder

WORKDIR /app

# Build arguments for environment variables
ARG REACT_APP_SERVER_URL
ARG REACT_APP_BACKEND_URL
ARG REACT_APP_SOCKET_URL

ENV REACT_APP_SERVER_URL=$REACT_APP_SERVER_URL
ENV REACT_APP_BACKEND_URL=$REACT_APP_BACKEND_URL
ENV REACT_APP_SOCKET_URL=$REACT_APP_SOCKET_URL
ENV NODE_OPTIONS="--max-old-space-size=4096"

COPY package*.json ./
RUN npm install --legacy-peer-deps

COPY . ./
RUN npm run build

# Stage 2: Serve with Nginx
FROM nginx:alpine

COPY --from=builder /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
