# 🎬 Media Stack – Docker Compose

Acest repository conține un **media stack complet** bazat pe Docker Compose:

* qBittorrent – download client
* Prowlarr – indexer manager
* Radarr – filme
* Sonarr – seriale
* Jellyfin – media server

Stack-ul este gândit să fie **simplu, stabil și ușor de extins**.

---

## 📁 Structura de directoare (host)

```text
.
├── docker-compose.yml
├── config/
│   ├── qbittorrent/
│   ├── radarr/
│   ├── sonarr/
│   ├── prowlarr/
│   └── jellyfin/
├── downloads/
│   ├── incomplete/
│   └── complete/
├── movies/
│   └── Movie Name (Year)/
└── tv/
    └── Show Name/
```

### 🔎 Explicații

* `config/` – configurațiile persistente ale containerelor
* `downloads/` – unde descarcă qBittorrent
* `movies/` – biblioteca finală de filme (Radarr)
* `tv/` – biblioteca finală de seriale (Sonarr)

> ⚠️ **Important**: toate containerele folosesc aceleași path-uri interne (`/downloads`, `/movies`, `/tv`).

---

## 🐳 Docker Compose

Pornește stack-ul cu:

```bash
docker compose up -d
```

Oprește-l cu:

```bash
docker compose down
```

---

## ⚙️ Configurare servicii

### 1️⃣ qBittorrent

* URL: [http://localhost:8080](http://localhost:8080)
* Rol: client de download

#### Setări recomandate

* **Options → Downloads**

  * Default Save Path: `/downloads/complete`
  * Keep incomplete torrents in: `/downloads/incomplete`
* **Options → Web UI**

  * Port: `8080`

#### Autentificare

* User: `admin`
* Parolă: `adminadmin` (schimb-o imediat, eu am gasit-o in log la container) 

---

### 2️⃣ Prowlarr

* URL: [http://localhost:9696](http://localhost:9696)
* Rol: management indexere (trackere)

#### Pași configurare

1. Settings → Indexers

   * adaugă trackerele dorite
2. Settings → Apps

   * adaugă **Radarr**

     * URL: `http://radarr:7878`
     * API Key: din Radarr
   * adaugă **Sonarr**

     * URL: `http://sonarr:8989`
     * API Key: din Sonarr

> Prowlarr NU descarcă nimic – doar trimite rezultate către *arr.

---

### 3️⃣ Radarr (Filme)

* URL: [http://localhost:7878](http://localhost:7878)
* Rol: management filme

#### Setări esențiale

**Media Management**

* Root Folder: `/movies`
* Enable Rename: ✅
* Movie Naming Format: implicit (sau custom)

**Download Clients**

* Type: qBittorrent
* Host: `qbittorrent`
* Port: `8080`
* Category: `movies`

**Completed Download Handling**

* Enable: ✅
* Remove completed: ❌ (recomandat)
* Use Hardlinks instead of Copy: ✅

---

### 4️⃣ Sonarr (Seriale)

* URL: [http://localhost:8989](http://localhost:8989)
* Rol: management seriale

#### Setări esențiale

**Media Management**

* Root Folder: `/tv`
* Enable Rename: ✅

**Download Clients**

* Type: qBittorrent
* Host: `qbittorrent`
* Port: `8080`
* Category: `tv`

**Completed Download Handling**

* Enable: ✅
* Use Hardlinks instead of Copy: ✅

---

### 5️⃣ Jellyfin

* URL: [http://localhost:8096](http://localhost:8096)
* Rol: media server (streaming)

#### Setup inițial

1. Creează user
2. Adaugă biblioteci:

   * Movies → `/media/movies`
   * TV Shows → `/media/tv`
3. Language: Romanian / English (după preferință)

> Jellyfin DOAR citește media, nu o modifică.

---

## 🔄 Workflow complet

1. Adaugi film / serial în Radarr / Sonarr
2. *arr caută prin Prowlarr
3. Torrentul ajunge în qBittorrent
4. Download în `/downloads`
5. Import automat în `/movies` sau `/tv`
6. Jellyfin vede conținutul și îl servește

---

## ✅ Best practices

* Folosește filesystem care suportă **hardlinks** (ext4)
* PUID / PGID identice pentru toate containerele
* Backup regulat la `config/`
* Schimbă parolele default

---

## 🔧 Extensii utile (opțional)

* **Bazarr** – subtitrări
* **Overseerr / Jellyseerr** – request UI
* **Nginx Proxy Manager** – reverse proxy + SSL
* **Watchtower** – auto update containere

---

Happy streaming 🍿
