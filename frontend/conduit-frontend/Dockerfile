FROM node:20-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .

ARG API_URL=http://localhost:8000/api
RUN sed -i "s#https://api.realworld.io/api#${API_URL}#g" \
    src/app/core/interceptors/api.interceptor.ts

RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist/angular-conduit /usr/share/nginx/html
EXPOSE 8282