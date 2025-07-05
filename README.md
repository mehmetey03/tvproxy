# tvproxy 📺

Un server proxy leggero e dockerizzato basato su **Flask** e **Requests**, progettato per superare restrizioni e accedere a flussi M3U/M3U8 senza interruzioni.

- 📥 **Scarica e modifica** flussi `.m3u` e `.m3u8` al volo.
- 🔁 **Proxa i segmenti** `.ts` mantenendo header personalizzati.
- 🚫 **Supera restrizioni** comuni come `Referer`, `User-Agent`, ecc.
- 🐳 **Facilmente dockerizzabile** su qualsiasi macchina, server o piattaforma cloud.
- 🧪 **Dashboard web completa** per amministrazione e monitoraggio in tempo reale.

---

## 📚 Indice

- Configurazione Autenticazione
- Configurazione per Server con 1 GB di RAM
- Piattaforme di Deploy
  - Render
  - HuggingFace
- Setup Locale
  - Docker
  - Termux (Android)
  - Python
- Dashboard di Amministrazione
- Utilizzo del Proxy
- Configurazione Proxy
- Gestione Docker

---

## 🔐 Configurazione Autenticazione (OBBLIGATORIA)

### Variabili d'Ambiente di Sicurezza

| Variabile        | Descrizione                                                         | Obbligatoria | Default       |
|------------------|---------------------------------------------------------------------|--------------|---------------|
| `ADMIN_PASSWORD` | Password per accedere alla dashboard di amministrazione            | **SÌ**       | `password123` |
| `SECRET_KEY`     | Chiave segreta per le sessioni Flask (deve essere univoca e sicura) | **SÌ**       | Nessuna       |
| `ADMIN_USERNAME` | Username per l'accesso (configurabile dalla web UI)                | No           | `admin`       |
| `ALLOWED_IPS`    | Lista di IP autorizzati separati da virgola                        | No           | Tutti gli IP  |

> ⚠️  **Obbligatorio**: impostare `ADMIN_PASSWORD` **e** `SECRET_KEY`.  
> 🔑 Usa un valore univoco per `SECRET_KEY`, ad esempio generato con:  
> `openssl rand -hex 32`  
> oppure:  
> `python -c 'import secrets; print(secrets.token_hex(32))'`

---

### 🐳 Esempio Docker

```bash
docker run -d -p 7860:7860 \
  -e ADMIN_PASSWORD=tua_password_sicura \
  -e SECRET_KEY=1f4d8e9a6c57bd2eec914d93cfb7a3efb9ae67f2643125c89cc3c50e75c4d4c3 \
  --name tvproxy tvproxy
```

---

### 📦 Esempio `.env` (Termux / Python)

```dotenv
ADMIN_PASSWORD=tua_password_sicura
SECRET_KEY=1f4d8e9a6c57bd2eec914d93cfb7a3efb9ae67f2643125c89cc3c50e75c4d4c3
```

---

## 💾 Configurazione per Server con RAM Limitata (1 GB)

### 📃 `.env` ottimizzato

```dotenv
# OBBLIGATORIO
ADMIN_PASSWORD=tua_password_sicura
SECRET_KEY=chiave_segreta_generata

# Ottimizzazioni memoria
REQUEST_TIMEOUT=30
KEEP_ALIVE_TIMEOUT=120
MAX_KEEP_ALIVE_REQUESTS=100
POOL_CONNECTIONS=5
POOL_MAXSIZE=10

# Cache ridotta
CACHE_TTL_M3U8=2
CACHE_TTL_TS=60
CACHE_TTL_KEY=60
CACHE_MAXSIZE_M3U8=50
CACHE_MAXSIZE_TS=200
CACHE_MAXSIZE_KEY=50
```

---

## 🚫 Disattivare la Cache per Streaming Diretto

  Se vuoi **disabilitare completamente la cache** (ad esempio per streaming diretto e contenuti sempre aggiornati), oi farlo aggiungendo questa riga al tuo file `.env` oppure dall`interfaccia web:

```
CACHE_ENABLED=False

```

---

## ☁️ Piattaforme di Deploy

### ▶️ Render

1. Projects → **New → Web Service** → *Public Git Repo*.
2. Repository: `https://github.com/nzo66/tvproxy` → **Connect**.
3. Scegli un nome, **Instance Type** `Free` (o superiore).
4. Aggiungi le variabili `ADMIN_PASSWORD` e `SECRET_KEY` nell'area **Environment**.
5. (Opzionale) Aggiungi `SOCKS5_PROXY`, `HTTP_PROXY`, `HTTPS_PROXY`.
6. **Create Web Service**.

### 🤖 HuggingFace Spaces

1. Crea un nuovo **Space** (SDK: *Docker*).
2. Carica `DockerfileHF` come `Dockerfile`.
3. Vai in **Settings → Secrets** e aggiungi `ADMIN_PASSWORD` e `SECRET_KEY`.
4. (Opzionale) Aggiungi `HTTP_PROXY` + `HTTPS_PROXY` (SOCKS5 non supportato su HF).
5. Dopo ogni modifica alle variabili fai **Factory Rebuild**.

#### **Configurazione Ottimizzata per HuggingFace**

Per **HuggingFace Spaces**, è consigliato utilizzare questa configurazione ottimizzata. Aggiungi le seguenti variabili nei **Secrets** del tuo Space:

```dotenv
# OBBLIGATORIO
ADMIN_PASSWORD=tua_password_sicura
SECRET_KEY=chiave_segreta_generata
CACHE_TTL_M3U8=5
CACHE_MAXSIZE_M3U8=500
CACHE_TTL_TS=600
CACHE_MAXSIZE_TS=8000
CACHE_TTL_KEY=600
CACHE_MAXSIZE_KEY=1000
POOL_CONNECTIONS=50
POOL_MAXSIZE=300
MAX_KEEP_ALIVE_REQUESTS=5000
KEEP_ALIVE_TIMEOUT=900
REQUEST_TIMEOUT=45
```

**Perché questa configurazione?**
- **Cache Ottimizzata**: Valori più elevati per gestire meglio i flussi video frequenti
- **Pool di Connessioni Potenziato**: Gestisce più connessioni simultanee nell'ambiente cloud
- **Timeout Bilanciati**: Equilibrio tra stabilità e performance per connessioni di lunga durata

---

## 🚫 Disattivare la Cache per Streaming Diretto

Se vuoi **disabilitare completamente la cache** (ad esempio per streaming diretto e contenuti sempre aggiornati), puoi farlo aggiungendo questa riga al tuo file `.env` oppure dall`interfaccia web:

```
CACHE_ENABLED=False

```

---

## 💻 Setup Locale

### 🐳 Docker

```bash
git clone https://github.com/nzo66/tvproxy.git
cd tvproxy
docker build -t tvproxy .

docker run -d -p 7860:7860 \
  -e ADMIN_PASSWORD=tua_password_sicura \
  -e SECRET_KEY=chiave_segreta_generata \
  --name tvproxy tvproxy
```

### 🐧 Termux (Android)

```bash
pkg update && pkg upgrade
pkg install git python nano -y

git clone https://github.com/nzo66/tvproxy.git
cd tvproxy
pip install -r requirements.txt

echo "ADMIN_PASSWORD=tua_password_sicura" > .env
echo "SECRET_KEY=chiave_segreta_generata" >> .env

gunicorn app:app -w 4 --worker-class gevent -b 0.0.0.0:7860
```

### 🐍 Python

```bash
git clone https://github.com/nzo66/tvproxy.git
cd tvproxy
pip install -r requirements.txt

echo "ADMIN_PASSWORD=tua_password_sicura" > .env
echo "SECRET_KEY=chiave_segreta_generata" >> .env

gunicorn app:app -w 4 --worker-class gevent --worker-connections 100 \
        -b 0.0.0.0:7860 --timeout 120 --keep-alive 5 \
        --max-requests 1000 --max-requests-jitter 100
```

---

## 🎛️ Dashboard di Amministrazione

- **🏠 Home**: `http://<server-ip>:7860/`
- **🔐 Login**: `http://<server-ip>:7860/login`
- **📊 Dashboard**: `http://<server-ip>:7860/dashboard`
- **⚙️ Config**: `http://<server-ip>:7860/admin/config`
- **📝 Log**: `http://<server-ip>:7860/admin/logs`
- **📈 API Stats**: `http://<server-ip>:7860/stats`

---

## 🧰 Utilizzo del Proxy

Sostituisci `<server-ip>` con l'indirizzo del tuo server.

### 💡 Liste M3U

```
http://<server-ip>:7860/proxy?url=<URL_LISTA_M3U>
```

### 📺 Flussi M3U8 con headers

```
http://<server-ip>:7860/proxy/m3u?url=<URL_FLUSSO_M3U8>&h_<HEADER>=<VALORE>
```

Esempio:
```
.../proxy/m3u?url=https://example.com/stream.m3u8&h_user-agent=VLC/3.0.20&h_referer=https://example.com/
```

### 🔍 Risoluzione DaddyLive 2025

```
http://<server-ip>:7860/proxy/resolve?url=<URL_DADDYLIVE>
```

### 🔑 Chiavi AES-128

```
http://<server-ip>:7860/proxy/key?url=<URL_CHIAVE>&h_<HEADER>=<VALORE>
```

---

## 🔁 Configurazione Proxy (Opzionale)

Se hai bisogno di escludere alcuni domini dall'utilizzo del proxy (ad esempio, per accessi diretti a servizi come `vavoo.to`), puoi usare la variabile `NO_PROXY_DOMAINS`.

| Variabile          | Descrizione                                                  | Esempio                                   |
|--------------------|--------------------------------------------------------------|-------------------------------------------|
| `SOCKS5_PROXY`     | Uno o più proxy SOCKS5, separati da virgola                  | `socks5://user:pass@host:port,...`        |
| `HTTP_PROXY`       | Proxy HTTP (usare in coppia con `HTTPS_PROXY`)               | `http://user:pass@host:port,...`          |
| `HTTPS_PROXY`      | Proxy HTTPS (di solito uguale a `HTTP_PROXY`)                | `http://user:pass@host:port,...`          |
| `NO_PROXY_DOMAINS` | Domini da escludere dal proxy, separati da virgola           | `github.com,vavoo.to`                     |

Esempio `.env`:

```dotenv
ADMIN_PASSWORD=tua_password_sicura
SECRET_KEY=chiave_segreta_generata
# SOCKS5_PROXY=socks5://user:pass@host1:1080
# HTTP_PROXY=http://user:pass@host:8080
# HTTPS_PROXY=http://user:pass@host:8080
```

In questo modo, le richieste verso `github.com` e `vavoo.to` non passeranno attraverso il proxy configurato, ma verranno eseguite direttamente.

---

## 🐳 Gestione Docker Rapida

```bash
docker logs -f tvproxy      # log in tempo reale
docker stop tvproxy         # ferma il container
docker start tvproxy        # avvia il container
docker rm -f tvproxy        # rimuovi il container
```

---

## ✅ Caratteristiche Principali

- Supporto automatico `.m3u` / `.m3u8`
- Headers personalizzati (`Authorization`, `Referer`, ...)
- Aggira restrizioni geografiche
- Compatibile con qualsiasi player IPTV
- Totalmente dockerizzato
- Dashboard web completa con statistiche, log, configurazioni
- Autenticazione sicura + whitelist IP
- Monitoraggio RAM / rete in tempo reale
- Cache intelligente M3U8 / TS / AES
- Configurazioni dinamiche **senza riavvio**

---

## 🎉 Enjoy the Stream!

> Goditi i tuoi flussi preferiti ovunque, senza restrizioni, con controllo completo e monitoraggio avanzato.
