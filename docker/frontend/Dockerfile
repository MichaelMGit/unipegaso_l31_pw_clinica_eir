# --- Fase 1: Build dell'applicazione ---
FROM node:20-alpine AS builder

RUN apk add --no-cache git bash

WORKDIR /app

ARG FRONTEND_REPO
ARG REACT_APP_API_URL

RUN git clone ${FRONTEND_REPO} .

ENV REACT_APP_API_URL=${REACT_APP_API_URL}

RUN if [ -f package-lock.json ]; then npm ci || npm install; else npm install; fi
RUN npm run build

FROM nginx:1.27-alpine

RUN rm /etc/nginx/conf.d/default.conf

COPY docker/frontend/nginx.conf /etc/nginx/conf.d/default.conf

COPY --from=builder /app/build /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]