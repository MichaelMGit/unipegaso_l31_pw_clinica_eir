# --- Fase 1: Build dell'applicazione ---
FROM node:20-alpine AS builder

# Installa git e bash per permettere la clonazione
RUN apk add --no-cache git bash

# Imposta la cartella di lavoro
WORKDIR /app

# Dichiara gli argomenti passati dal docker-compose.yml
ARG FRONTEND_REPO
ARG REACT_APP_API_URL

# Clona il repository direttamente nella cartella corrente (/app)
RUN git clone ${FRONTEND_REPO} .

# Esporta la variabile d'ambiente affinché React la legga durante la build
ENV REACT_APP_API_URL=${REACT_APP_API_URL}

# Installa le dipendenze e compila l'applicazione
RUN if [ -f package-lock.json ]; then npm ci || npm install; else npm install; fi
RUN npm run build

# --- Fase 2: Server Nginx per la produzione ---
FROM nginx:1.27-alpine

# Rimuove la configurazione di default di Nginx
RUN rm /etc/nginx/conf.d/default.conf

# Copia il tuo nginx.conf per gestire correttamente il routing della Single Page Application
COPY docker/frontend/nginx.conf /etc/nginx/conf.d/default.conf

# Copia i file statici compilati dalla fase precedente
COPY --from=builder /app/build /usr/share/nginx/html

# Espone la porta 80
EXPOSE 80

# Avvia Nginx in primo piano
CMD ["nginx", "-g", "daemon off;"]