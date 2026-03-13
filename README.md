# Stack Docker: React + FastAPI + MySQL

Questo workspace contiene i file necessari per avviare una stack composta da:

- frontend React, clonato dal repository indicato in `FRONTEND_REPO`
- backend FastAPI, clonato dal repository indicato in `BACKEND_REPO`
- database MySQL 8

## File inclusi

- `.env`: variabili centralizzate per database e applicazioni
- `.env.example`: template versionabile senza segreti reali
- `docker-compose.yml`: orchestra i tre servizi
- `docker/frontend/Dockerfile`: immagine base per il frontend React
- `docker/backend/Dockerfile`: immagine base per il backend FastAPI
- `docker/backend/referti/`: archivio locale copiato in `uploads/referti` del backend clonato
- `mysql/init/pw_l31_sanita.sql`: dump completo del database da importare al primo avvio

## Comportamento

Alla prima esecuzione:

- l'immagine `frontend` clona automaticamente il repository FE durante la build Docker
- il frontend esegue `npm run build` durante la build Docker e poi serve i file statici con Nginx
- l'immagine `backend` clona automaticamente il repository BE durante la build Docker
- dopo il clone del backend, i file presenti in `docker/backend/referti/` vengono copiati in `uploads/referti`
- il backend installa le dipendenze in build e si avvia in modalità production senza `--reload`
- MySQL crea database e utente applicativo tramite le variabili `MYSQL_DATABASE`, `MYSQL_USER` e `MYSQL_PASSWORD`
- MySQL importa il dump completo `mysql/init/pw_l31_sanita.sql`
- il backend parte solo dopo che MySQL risponde in healthcheck, quindi dopo il bootstrap iniziale del database

Alle esecuzioni successive frontend e backend vengono ricostruiti solo quando rilanci la build Docker.

Importante: gli script in `mysql/init/` vengono eseguiti da MySQL solo quando il volume dati è vuoto. Se aggiorni il dump o vuoi ricaricare il database da zero, devi rimuovere il volume MySQL prima del nuovo avvio.

In questo setup la cartella `mysql/init/` contiene solo il dump completo: la creazione iniziale di database e utente è demandata direttamente all'entrypoint ufficiale di MySQL tramite le variabili d'ambiente.

## Porte esposte

- Frontend production: `http://localhost:3000`
- FastAPI: `http://localhost:8001`
- MySQL: `localhost:3306`
- Swagger UI FastAPI: `http://localhost:8001/docs`

## Credenziali database di default

- database: `pw_l31_sanita`
- utente: `pw_l31`
- password: `jehJnez6.D4R(s!A`
- root password: `root`

## Variabili d'ambiente applicative

Le variabili sono centralizzate nel file `.env`, già pronto in root progetto.

Il file `.env.example` contiene un template sicuro da versionare e usare come base per nuovi ambienti.

### Backend FastAPI

- `DATABASE_URL=mysql+pymysql://pw_l31:jehJnez6.D4R(s!A@mysql:3306/pw_l31_sanita`
- `JWT_SECRET=change-me-jwt-secret`
- `CORS_ORIGINS=http://localhost:3000`
- `BACKEND_REPO=https://github.com/MichaelMGit/unipegaso_l31_be.git`

### Frontend React

- `REACT_APP_API_URL=http://127.0.0.1:8001`
- `FRONTEND_REPO=https://github.com/MichaelMGit/unipegaso_l31_fe.git`

## Comandi Docker Compose

Se vuoi cambiare credenziali, secret o porte, modifica prima il file `.env`.

Se devi ricreare il file locale da zero, copia `.env.example` in `.env` e poi personalizza i valori.

### Avvio in foreground

```powershell
docker compose up --build
```

### Avvio in background (detached)

```powershell
docker compose up --build -d
```

### Vedere lo stato dei container

```powershell
docker compose ps
```

### Vedere i log

Tutti i servizi:

```powershell
docker compose logs -f
```

Solo backend:

```powershell
docker compose logs -f backend
```

Solo MySQL:

```powershell
docker compose logs -f mysql
```

### Fermare la stack

```powershell
docker compose down
```

### Fermare la stack e rimuovere i volumi

```powershell
docker compose down -v
```

Usa questo comando quando vuoi forzare una nuova importazione del dump `mysql/init/pw_l31_sanita.sql` al successivo avvio.

### Ricostruire senza cache

```powershell
docker compose build --no-cache
docker compose up -d
```

### Ripartenza pulita completa

```powershell
docker compose down -v
docker compose up --build
```

## Note importanti

### Frontend React

Il frontend è configurato in modalità production:

- clona il repository durante la build immagine
- installa le dipendenze
- esegue `npm run build`
- serve l'output statico tramite Nginx

Nginx usa una configurazione custom con fallback SPA verso `index.html`, così rotte frontend come `/login` funzionano correttamente anche con refresh diretto del browser.

Questa configurazione presuppone che il progetto generi la cartella `build/`, tipica di Create React App.

Se il repository usa Vite o genera `dist/`, andrà adattato `docker/frontend/Dockerfile` nel `COPY --from=builder` finale.

### Backend FastAPI

Il backend è configurato in modalità production:

- clona il repository durante la build immagine
- installa le dipendenze Python durante la build
- avvia Uvicorn senza `--reload`

Il container prova questi entrypoint, in quest'ordine:

1. `main.py` come `main:app`
2. `app/main.py` come `app.main:app`

Se il repository backend usa una struttura diversa, bisognerà aggiornare il comando del servizio `backend` in `docker-compose.yml`.

Con il setup attuale non c'è più bisogno che il backend esegua seed iniziali del database: i dati arrivano direttamente dal dump MySQL importato in fase di bootstrap.

Il bootstrap del backend gira in shell Linux del container, non dipende dalla shell Windows del tuo host.

FastAPI e React sono collegati sulla stessa rete Docker insieme a MySQL, quindi possono comunicare direttamente usando i nomi dei container (`clinica_mysql`, `clinica_backend`, `clinica_frontend`).

Nel setup attuale, però, il browser chiama il backend tramite `http://127.0.0.1:8001`, come definito in `REACT_APP_API_URL`.

### Dipendenze backend

L'immagine installa automaticamente `requirements.txt` se presente. Se il progetto usa `pyproject.toml`, Poetry o un'altra struttura, conviene adattare il Dockerfile del backend.

## Possibili miglioramenti

Se vuoi, nel prossimo passo posso anche prepararti:

- un file `.env` per centralizzare le variabili
- profili `dev`/`prod`
- un reverse proxy Nginx
- healthcheck applicativi per frontend e backend
- mount locali dei sorgenti invece del clone dentro i container
