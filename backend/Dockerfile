FROM node:18-alpine

WORKDIR /app

COPY package.json ./
RUN npm install

COPY . .

EXPOSE 4000

# default redis host (local docker)
ENV REDIS_HOST=redis

CMD ["node","server.js"]
