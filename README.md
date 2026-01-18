
# Dockerumgebung – Infrastruktur & Automation

Dieses Repository bündelt eine komplette lokale/produktive Docker‑Umgebung für Automatisierung (n8n), PDF‑Extraktion (pdf2text), PlantUML und Cloudflare‑Tunneling. Zusätzlich enthält es Dokumente und Workflows für KI‑gestützte Hinweise/Tipps.

## Überblick

- **Infrastruktur‑Templates**: wiederverwendbare Docker‑Compose‑Vorlagen für Services.
- **Produktiv‑Stack**: lauffähige n8n‑Instanz mit Traefik, Let's Encrypt und integrierter pdf2text‑API.
- **Diagramme**: PlantUML‑Dateien zur Netzwerk‑ und Service‑Architektur.
- **KI‑Hilfen**: Templates und Zusammenfassungen im Ordner HintsundTipps‑mit‑KI.

## Projektstruktur

```
.
├── HintsundTipps-mit-KI/        # KI-Dokumente, Hinweise & Templates
├── infrastructure/              # Infrastruktur-Templates
│   ├── services/                # Docker Services (n8n, cloudflared, plantuml, pdf2text)
│   ├── network-architecture.puml
│   └── services-details.puml
└── n8n-compose/                  # Produktive n8n-Compose-Instanz + Daten
```

## Services (Templates)

Die Service‑Vorlagen befinden sich unter infrastructure/services/:

- **n8n**: Automationsplattform mit mehreren Compose‑Varianten
- **pdf2text**: FastAPI‑Service zur PDF‑Text‑Extraktion
- **plantuml**: PlantUML Diagram Server
- **cloudflared**: Cloudflare Tunnel (Quick Tunnel & Full Setup)

Details und Kommandos siehe [infrastructure/services/README.md](infrastructure/services/README.md).

## Produktive n8n‑Instanz

Die produktive Compose‑Konfiguration liegt unter n8n-compose/:

- n8n + Traefik + Let's Encrypt
- integrierter pdf2text‑Service
- lokale Daten und Credentials in local-files/ (nicht committiert)

Details und Kommandos siehe [n8n-compose/README.md](n8n-compose/README.md).

## Schnellstart (lokal)

1. In das gewünschte Service‑Verzeichnis wechseln.
2. Docker Compose starten.

Beispiele:

- [infrastructure/services/n8n/](infrastructure/services/n8n/)
- [infrastructure/services/pdf2text/](infrastructure/services/pdf2text/)
- [n8n-compose/](n8n-compose/) (produktiver Stack)

## Architektur‑Diagramme

- Netzwerkübersicht: [infrastructure/network-architecture.puml](infrastructure/network-architecture.puml)
- Service‑Details: [infrastructure/services-details.puml](infrastructure/services-details.puml)

Hinweis: GitHub rendert PlantUML nicht nativ. Deshalb sind die Diagramme hier als SVG eingebettet.

### Netzwerkübersicht (SVG)

![Netzwerkübersicht](infrastructure/network-architecture.svg)

### Service‑Details (SVG)

![Service-Details](infrastructure/services-details.svg)

## Hinweise zu Secrets & Daten

- .env‑Dateien sind lokal und werden nicht committiert.
- local-files/ enthält produktive n8n‑Daten und ist zu schützen.

## GitHub Copilot Instructions

Dieses Repository enthält projektspezifische Regeln für GitHub Copilot in [.github/copilot-instructions.md](.github/copilot-instructions.md).

Enthalten sind u.a.:

- Projektziele und Arbeitsweise (kleine, zielgerichtete Änderungen; reproduzierbare Schritte lokal/CI)
- Hinweise zu generierten Dateien/Indizes (z.B. Prompts-/AwesomeCopilot-READMEs) und dass diese nicht „vergessen“ werden sollen
- Referenzen auf CI-/Generator-Workflows (welche Generatoren laufen sollen und wie Unit-Tests per `unittest` ausgeführt werden)
- Qualitäts-/Markdown-Regeln sowie der Hinweis, keine Secrets/Keys ins Repo zu schreiben

## Release & Package

Dieses Repository kann per GitHub Actions als **GitHub Release** paketiert werden (ZIP + TAR.GZ + SHA256 Checksums). Details siehe [RELEASE.md](RELEASE.md).

Release (Assets) erstellen:

- Tag erstellen: `git tag -a v1.0.0 -m "v1.0.0"`
- Tag pushen: `git push origin v1.0.0`

Der Workflow [.github/workflows/release.yml](.github/workflows/release.yml) erstellt dann automatisch einen GitHub Release und hängt die Assets an.

GitHub **Packages** (Container Images) werden separat nach GHCR veröffentlicht:

- Workflow: [.github/workflows/publish-ghcr.yml](.github/workflows/publish-ghcr.yml)
- Images:
	- `ghcr.io/<owner>/<repo>/pdf2text`
	- `ghcr.io/<owner>/<repo>/plantuml`

Hinweis: Git Tag `v1.0.0` wird als Container-Tag `1.0.0` veröffentlicht (zusätzlich auch `latest`).

Falls du `v1.0.0` bereits gepusht hattest, bevor der Workflow existierte: Actions → "Publish GHCR Images" → "Run workflow" und als `version` z.B. `1.0.0` angeben.

## Rollenfokus

Diese Umgebung ist auf einen **Full‑Stack‑DevOps‑Workflow** ausgelegt: Infrastruktur‑Templates, produktiver Automations‑Stack, APIs und Dokumentation in einem Repo.
