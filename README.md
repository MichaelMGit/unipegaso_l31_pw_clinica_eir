# Project Work UniPegaso L31 - Tema n. 1, Traccia 16

Questo repository raccoglie l'infrastruttura necessaria all'esecuzione del project work del corso di laurea **UniPegaso L31**, sviluppato nell'ambito del:

- **Tema n. 1:** La digitalizzazione dell’impresa
- **Traccia 16:** Sviluppo di una applicazione full-stack API-based per un’organizzazione del settore sanitario

La finalità del progetto è predisporre un ambiente di avvio semplice, riproducibile e coerente con l'architettura dell'applicazione, così da consentire l'esecuzione coordinata dei componenti frontend, backend e database mediante container Docker.

## Finalità del repository

Questo repository non contiene direttamente l'intero codice applicativo del frontend e del backend, ma fornisce l'ambiente di orchestrazione necessario per:

- clonare i repository applicativi;
- configurare ed eseguire i container della soluzione;
- inizializzare il database MySQL a partire da un dump completo;
- caricare i referti di esempio nel backend;
- avviare l'intera architettura full-stack in modo coordinato.

## Architettura della soluzione

La stack è composta dai seguenti elementi:

- **frontend React**, clonato dal repository indicato nella variabile `FRONTEND_REPO`;
- **backend FastAPI**, clonato dal repository indicato nella variabile `BACKEND_REPO`;
- **database MySQL 8**, inizializzato automaticamente tramite dump.

Il repository include inoltre:

- `docker-compose.yml`, per l'orchestrazione dei servizi;
- `docker/frontend/Dockerfile`, per la build del frontend;
- `docker/backend/Dockerfile`, per la build del backend;
- `docker/backend/referti/`, contenente i file da copiare nella cartella `uploads/referti` del backend clonato;
- `mysql/init/pw_l31_sanita.sql`, contenente il backup completo del database;
- `.env`, già predisposto con le variabili necessarie all'avvio della stack;
- `.env.example`, mantenuto come template di riferimento.

## Configurazione iniziale

Nel repository è già presente un file `.env` pronto all'uso. Per il primo avvio, pertanto, **non è necessario creare manualmente alcun file di configurazione**.

All'interno del file sono già definiti:

- le credenziali di accesso a MySQL;
- gli URL dei repository backend e frontend;
- la variabile `DATABASE_URL` utilizzata dal backend FastAPI;
- le porte esposte dai servizi;
- le principali variabili applicative richieste dall'ambiente Docker.

Qualora si renda necessario personalizzare credenziali, porte o repository sorgente, è sufficiente modificare direttamente il file `.env` prima dell'avvio della stack.

## Bootstrap dell'applicazione

Al primo avvio dell'ambiente Docker, il processo di bootstrap avviene nel modo seguente:

1. MySQL crea il database e l'utente applicativo utilizzando le variabili `MYSQL_DATABASE`, `MYSQL_USER` e `MYSQL_PASSWORD`;
2. MySQL importa automaticamente il dump contenuto in `mysql/init/pw_l31_sanita.sql`;
3. il backend viene clonato dal repository definito in `BACKEND_REPO`;
4. i file presenti nella cartella `docker/backend/referti/` vengono copiati nella directory `uploads/referti` del backend;
5. il backend viene avviato soltanto dopo che il database risulta disponibile;
6. il frontend viene clonato, compilato e pubblicato tramite Nginx.

## Avvio della stack

Dalla root del progetto è possibile avviare l'ambiente con il comando:

```powershell
docker compose up --build
```

Per l'avvio in background:

```powershell
docker compose up --build -d
```

## Monitoraggio dei servizi

Per verificare lo stato dei container:

```powershell
docker compose ps
```

Per visualizzare i log di tutti i servizi:

```powershell
docker compose logs -f
```

Per visualizzare i log del solo backend:

```powershell
docker compose logs -f backend
```

Per visualizzare i log del solo database MySQL:

```powershell
docker compose logs -f mysql
```

## Arresto e ripristino dell'ambiente

Per arrestare la stack:

```powershell
docker compose down
```

Per arrestarla rimuovendo anche i volumi Docker:

```powershell
docker compose down -v
```

Quest'ultima opzione è utile quando si desidera forzare una nuova importazione del dump MySQL al successivo avvio.

Per una ripartenza completa da zero:

```powershell
docker compose down -v
docker compose up --build
```

Per ricostruire le immagini senza utilizzare la cache:

```powershell
docker compose build --no-cache
docker compose up -d
```

## Porte esposte

- Frontend: `http://localhost:3000`
- Backend FastAPI: `http://localhost:8001`
- Documentazione Swagger: `http://localhost:8001/docs`
- MySQL: `localhost:3306`

## Credenziali demo

Per agevolare le attività di prova e validazione funzionale dell'applicazione, il database inizializzato tramite dump mette a disposizione i seguenti account demo:

- `mario.rossi@clinicaeir.it` / `medico123`
- `laura.bianchi@clinicaeir.it` / `medico123`
- `giovanni.verdi@example.com` / `paziente123`
- `segreteria@clinicaeir.it` / `segreteria123`
- `admin@clinicaeir.it` / `admin123`

## Considerazioni operative

- Il file `.env` è già fornito e pronto all'uso;
- il dump MySQL viene importato solamente quando il volume dati risulta vuoto;
- i repository del frontend e del backend vengono determinati tramite variabili d'ambiente;
- i referti presenti in `docker/backend/referti/` vengono copiati nel backend durante la build dell'immagine;
- il backend non richiede più un seed iniziale applicativo, poiché i dati vengono caricati dal dump del database.

## Repository applicativi di supporto

I repository sorgente utilizzati dall'ambiente sono configurati tramite le seguenti variabili:

- `BACKEND_REPO`
- `FRONTEND_REPO`

I relativi valori di default sono già presenti nel file `.env` incluso nel progetto.
