# 📋 EIC Agent - Projekt Setup & Wartung

## 🏗️ Infrastruktur Übersicht

### Ihre aktuelle Konstellation:
```
Mac (Entwicklung)         GitHub Repository        Hetzner Server (Produktion)
localhost                →  MaxSeizo/eic-agent   →  ai.eic-insurance.de
VSCode                      (Versionskontrolle)     162.55.188.206
```

## 🎯 Wichtig zu verstehen

### Was ist EIC Agent?
- **Frontend**: Ihr eigenes Branding (Logos, Texte, Farben)
- **Backend**: Original Open WebUI (100% Funktionalität erhalten)
- **Prinzip**: Nur visuelle Anpassungen, keine Funktionsänderungen

### Warum diese Trennung?
- Updates von Open WebUI können einfach übernommen werden
- Backend-Funktionen bleiben stabil
- Nur Frontend wird angepasst (sicherer Ansatz)

## 🖥️ Server Setup (Hetzner)

### Was läuft auf dem Server:
```bash
# Docker Container mit Open WebUI
Container: open-webui
Image: ghcr.io/open-webui/open-webui:main
Port: 3080 → 8080
Verwaltung: docker-compose
```

### Wichtige Dateien auf dem Server:
```
/root/eic-agent/
├── docker-compose.yml       # Container-Konfiguration
├── override-branding.sh     # Script für Branding-Anpassungen
├── static/static/          # Ihre Logo-Dateien
└── src/                    # Frontend-Code (wird nicht verwendet)
```

## 🔄 Deployment Workflow

### 1. Lokale Entwicklung (Mac)
```bash
cd ~/Development/Projects/eic-agent
code .  # VSCode öffnen

# Änderungen machen:
# - Logos: static/static/
# - Texte: src/routes/+layout.svelte
# - App Name: src/lib/constants.ts
```

### 2. Zu GitHub pushen
```bash
git add .
git commit -m "Beschreibung der Änderung"
git push
```

### 3. Auf Server deployen
```bash
# SSH zum Server
ssh root@162.55.188.206

# Updates holen
cd /root/eic-agent
git pull

# Container neu starten (behält alle Daten)
docker-compose restart

# Oder komplett neu bauen (verliert Daten!)
docker-compose down
docker-compose up -d
```

## 🎨 Was kann angepasst werden?

### ✅ Kann geändert werden (Frontend):
- **Logos & Favicons**: `static/static/`
  - logo.png
  - favicon.png
  - splash.png (Login-Screen)
  - apple-touch-icon.png
  
- **App-Name**: Wird automatisch überschrieben zu "EIC Agent"
- **Farben**: `src/app.css` (aktuell nicht aktiv)
- **Texte**: In Svelte-Komponenten

### ❌ Sollte NICHT geändert werden:
- Backend-Code (`backend/`)
- API-Endpoints
- Datenbank-Struktur
- Docker-Konfiguration (außer docker-compose.yml)

## 🐳 Docker-Compose Erklärung

### docker-compose.yml:
```yaml
services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main  # Offizielles Image
    container_name: open-webui
    ports:
      - "3080:8080"  # Port 3080 extern, 8080 intern
    volumes:
      - open-webui-data:/app/backend/data  # Persistent Daten
      - ./override-branding.sh:/app/override-branding.sh  # Branding Script
      - ./static/static:/app/custom-static  # Ihre Logos
    command: bash -c "/app/override-branding.sh && bash start.sh"  # Start mit Branding
```

### override-branding.sh:
Dieses Script läuft beim Container-Start und:
1. Kopiert Ihre Logos an die richtigen Stellen
2. Ersetzt "Open WebUI" mit "EIC Agent" in allen Dateien

## 🔧 Häufige Befehle

### Container-Status prüfen:
```bash
docker ps                    # Läuft der Container?
docker logs open-webui       # Logs anschauen
docker-compose logs -f       # Live-Logs verfolgen
```

### Container neu starten:
```bash
cd /root/eic-agent
docker-compose restart       # Schneller Neustart
docker-compose down && docker-compose up -d  # Kompletter Neustart
```

### Logos aktualisieren:
```bash
# Auf Mac: Logos in static/static/ ersetzen
git add . && git commit -m "Neue Logos" && git push

# Auf Server:
ssh root@162.55.188.206
cd /root/eic-agent
git pull
docker-compose restart
```

### Bei Problemen:
```bash
# Container komplett neu bauen
docker-compose down
docker-compose pull
docker-compose up -d

# Logs prüfen
docker-compose logs --tail 50
```

## 📝 Wichtige Hinweise

### Datenbank & Benutzerdaten:
- Gespeichert in Docker Volume `open-webui-data`
- Bleibt bei `docker-compose restart` erhalten
- Wird gelöscht bei `docker-compose down -v` (Vorsicht!)

### Browser-Cache:
- Nach Änderungen IMMER Cache leeren
- Mac: Cmd+Shift+R oder Inkognito-Modus
- Manchmal hilft: URL mit `?v=2` anhängen

### Updates von Open WebUI:
```bash
# Neuste Version holen
docker-compose pull
docker-compose up -d
# Ihre Anpassungen werden automatisch wieder angewendet!
```

## 🚨 Troubleshooting

### "Open WebUI" wird noch angezeigt:
1. Browser-Cache komplett leeren
2. Container neu starten: `docker-compose restart`
3. Warten Sie 30 Sekunden nach Neustart

### Logos werden nicht angezeigt:
1. Prüfen ob Dateien vorhanden: `ls -la static/static/`
2. Container neu starten
3. Browser-Cache leeren

### 502 Bad Gateway:
```bash
docker ps  # Läuft Container?
docker-compose up -d  # Container starten
```

### Ollama Connection Error:
- Normal wenn Ollama nicht installiert
- In Admin-Panel konfigurieren oder ignorieren

## 🔐 Sicherheit

### Wichtig:
- Ändern Sie NIE das Backend
- Committen Sie keine Passwörter oder API-Keys
- Backups: `docker exec open-webui tar -czf /backup.tar.gz /app/backend/data`

## 📞 Support-Workflow

Bei Problemen:
1. Logs prüfen: `docker-compose logs --tail 100`
2. Container-Status: `docker ps`
3. GitHub auf aktuellem Stand? `git status`
4. Browser-Cache geleert?

## 🎯 Zusammenfassung

**Ihr Setup in einem Satz:**
Sie nutzen das Original Open WebUI Backend (volle Funktionalität) mit Ihrem eigenen Frontend-Branding (EIC Agent), deployed via Docker-Compose auf Hetzner.

**Golden Rule:**
Backend nie anfassen, nur Frontend anpassen! 

**Deployment:**
Mac → GitHub → Hetzner Server → Docker-Compose → Live

---
*Erstellt für das EIC Agent Projekt - Stand: August 2024*