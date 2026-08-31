FROM node:18-slim

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install production dependencies
RUN npm install --production

# Copy application source
COPY . .

# Expose socket server port
EXPOSE 4000

ENV NODE_ENV=production
ENV PORT=4000

CMD ["node", "index.js"]
