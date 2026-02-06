#build stage 
FROM node:18-alpine AS build

WORKDIR /app

# Copy package files 
COPY package.json package-lock.json ./

# Install dependencies
RUN npm install

# Copy source code
COPY . .

# Build 
RUN npm run build

# stage 2 light weight image 
FROM nginx:alpine

# Copy built React app
COPY --from=build /app/build /usr/share/nginx/html


EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]

