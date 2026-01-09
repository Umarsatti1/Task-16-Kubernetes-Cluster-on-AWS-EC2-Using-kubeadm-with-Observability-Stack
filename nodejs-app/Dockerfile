# Use Node.js official image
FROM node:20-alpine

# Set working directory
WORKDIR /usr/src/app

# Copy package.json first for caching
COPY package.json ./

# Install dependencies
RUN npm install

# Copy rest of the app
COPY . .

# Expose port
EXPOSE 3000

# Start the app
CMD ["npm", "start"]