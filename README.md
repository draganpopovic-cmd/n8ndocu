# n8n Automation Platform (Produktive Instanz)

Dies ist die **aktive Produktionsumgebung** für n8n mit Traefik, Let's Encrypt und integriertem pdf2text Service.

## Verzeichnisstruktur

```text
n8n-compose/
├── docker-compose.yml       # Produktive Konfiguration
├── .env                      # Umgebungsvariablen (lokal, wird nicht committed)
├── .env.cloudflare.example   # Cloudflare Tunnel Vorlage
├── local-files/              # n8n Daten & Credentials (🔒 nicht committiert)
│   └── n8n-data/
├── n8n-files/                # Custom Nodes & Extensions
└── pdf2text/                 # PDF Extraction Service
    ├── Dockerfile
    └── app.py
```

## Services

### n8n

- **Port:** `127.0.0.1:5678` (lokal)
- **Domain:** `${SUBDOMAIN}.${DOMAIN_NAME}` (via Traefik)
- **Daten:** `./local-files/n8n-data/` (verschlüsselt)
- **Custom Nodes:** `./n8n-files/`

### pdf2text

- **Port:** `8001` (intern: 8000)
- **Build:** Lokales Dockerfile aus `./pdf2text/`
- **API:** `http://localhost:8001` oder via Traefik

### Traefik

- **HTTPS:** Automatisch via Let's Encrypt
- **Port:** `80`, `443`
- **API:** `http://localhost:8080/api` (insecure mode)

## Commands

```bash
# Starten
docker compose up -d

# Status
docker compose ps
docker compose logs -f

# Nur pdf2text Logs
docker compose logs -f pdf2text

# Stoppen
docker compose down

# Mit Cloudflare Tunnel (Alternative)
docker compose -f infrastructure/services/n8n/docker-compose.cloudflare.yml up -d
```

## Konfiguration

Lokale `.env` wird **nicht committiert** - siehe [.gitignore](../.gitignore)

**Wichtige Variablen:**

- `DOMAIN_NAME` - Top-Level Domain (z.B. example.ai)
- `SUBDOMAIN` - n8n Subdomain (z.B. n8n)
- `GENERIC_TIMEZONE` - Timezone für Workflows (z.B. Europe/Berlin)
- `SSL_EMAIL` - Email für Let's Encrypt Certificate
- `PDF2TEXT_PORT` - Host-Port für pdf2text (Standard: 8001)

## Templates

Die aktuellen Configurations-Templates sind in:

- `infrastructure/services/n8n/docker-compose.yml` - Hauptversion
- `infrastructure/services/n8n/docker-compose.cloudflare.yml` - Cloudflare Variant
- `infrastructure/services/n8n/docker-compose.quick-tunnel.yml` - Quick Tunnel Variant

Änderungen hier sollten in `n8n-compose/docker-compose.yml` **manuell aktualisiert** werden.

## Sicherheit

⚠️ **WICHTIG:**

- `local-files/` enthält Credentials und ist zu schützen (nur lokal)
- `.env` wird nicht committiert (siehe `.gitignore`)
- Produktionsdaten müssen regelmäßig gebackupt werden

## Workflows (JSON Exporte)

Im Ordner liegen mehrere exportierte n8n-Workflows (`*.json`). Du kannst sie in n8n über **Workflows → Import from File** importieren.

### Voraussetzungen

- Der Stack läuft (siehe Commands oben), damit interne Hosts wie `http://pdf2text:8000` auflösbar sind.
- Das Volume/Bind-Mount für `/home/node/.n8n-files/` ist korrekt gemountet, wenn Workflows mit lokalen Dateien arbeiten.
- Credentials müssen nach dem Import ggf. neu zugewiesen werden (IDs in JSON sind nur Platzhalter bzw. installationsspezifisch).

### Enthaltene Workflows

- `Automatisierte Lead-Qualifizierung.json` — IMAP Mail-Trigger → KI-Qualifizierung (OpenAI) → Folgeaktionen; benötigt IMAP- und OpenAI-Credentials.
- `PDF -_ Text (Sidecar) -_ OpenAI Summary -_ Datei.json` — PDF → Text via `pdf2text` → OpenAI Summary → als Datei speichern; benötigt `pdf2text`, OpenAI-Credentials, Zugriff auf `/home/node/.n8n-files/`.
- `readpdfandsummarizewithopenai.json` — Liest PDFs aus `/home/node/.n8n-files/*.pdf`, fasst sie zusammen und speichert Ergebnisse; benötigt OpenAI-Credentials.
- `tokenberechnen.json` — Beispiel für OpenAI Token-/Kosten-Tracking (Responses API) inkl. Preis-Mapping pro 1M Tokens; benötigt `OPENAI_API_KEY` (Preise im Workflow hart codiert).
- `Drive PDFs -_ Summaries -_ README.md (Full Index).json` und `googledrive.json` — Google Drive PDFs iterieren → zusammenfassen → Index/README-artige Ausgabe; benötigt Google-Drive-OAuth2 + OpenAI-Credentials.
- `listonedrivefiles.json` — Listet Google-Drive-Ordnerinhalte und formatiert die Ausgabe; benötigt Google-Drive-OAuth2.
- `pdf2textdockerinstall.json` — Variante/Experiment rund um PDF→Text über `pdf2text` (teils abweichender Port/URL); nach Import `url`/Port prüfen.

### Typische Nacharbeit nach dem Import

- In jedem OpenAI/HTTP/Drive/IMAP Node die Credentials neu auswählen.
- Dateipfade (z.B. `/home/node/.n8n-files/PDF/...`) an deine Ordnerstruktur anpassen.
- Bei `pdf2text` Nodes die URL auf den tatsächlich genutzten Service/Port prüfen (im Compose meist `http://pdf2text:8000/extract`).

