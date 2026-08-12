FROM node:20-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .

ARG API_URL=http://localhost:8000/api
RUN sed "s#__API_URL__#${API_URL}#g" \
    src/environments/environment.prod.template.ts \
    > src/environments/environment.prod.ts

RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist/angular-conduit /usr/share/nginx/html
EXPOSE 8282