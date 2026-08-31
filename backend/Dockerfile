FROM node:18-slim

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install production dependencies
RUN npm install --production

# Copy application source
COPY . .

# Expose backend REST API port
EXPOSE 8000

ENV NODE_ENV=production
ENV PORT=8000

CMD ["node", "server.js"]
