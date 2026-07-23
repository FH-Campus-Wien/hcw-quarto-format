# HCW Quarto Format

Gemeinsames PDF-Format der Hochschule Campus Wien für Quarto-Dokumente. Dieses
Repository ist die zentrale Quelle für die visuelle Gestaltung aller
HCW-Dokumentvorlagen.

## Enthalten

- HCW-Logo auf der Titelseite
- A4-Seitenlayout
- TeX Gyre Heros als freie, Helvetica-kompatible Standardschrift
- einheitliche Überschriften, Bild- und Tabellenbeschriftungen
- Seitenkopf und Seitennummerierung
- generische Felder für Titel, Untertitel, Dokumenttyp, Autor:innen, Version
  und Datum

Das Format heißt `hcw-pdf`. Eine Dokumentvorlage aktiviert es beispielsweise
so:

```yaml
format:
  hcw-pdf:
    toc: true
    number-sections: true
```

## Abgrenzung zu Dokumentvorlagen

Das Format legt die gemeinsame Darstellung fest. Fachliche Struktur und
Inhalte bleiben in eigenen Vorlagen.

| Gehört in `hcw-quarto-format` | Bleibt in der jeweiligen Vorlage |
|---|---|
| Logo und Titelseiten-Grundlayout | Kapitel- oder Seitenstruktur |
| Schrift, Farben und Seitenränder | Beispieltexte und Screenshots |
| Kopf-/Fußzeilen und Seitennummern | Dokumentartspezifische Metadaten |
| Bild- und Tabellenbeschriftungen | Literaturdatenbank |
| gemeinsame LaTeX-Konfiguration | Erklärungen und Pflichttexte |

Beim Thesis-Template bleiben daher insbesondere Studiengang, Art der
Abschlussarbeit, Betreuung, Kurzfassung, Abstract, Eigenständigkeitserklärung,
Kapitelstruktur und Literaturbeispiele im Thesis-Repository.

## Repository-Struktur

```text
_extensions/hcw/
├── _extension.yml
├── header.tex
├── title.tex
└── assets/images/hcw-logo-anthrazit.png
template.qmd
```

`template.qmd` ist ein visuelles Testdokument für die Entwicklung des Formats.
Ein lokal abgelegtes Word-Template kann als Gestaltungsreferenz dienen, ist
aber nicht Teil der veröffentlichten Quarto-Erweiterung.

## Lokal testen

Im Repository:

```bash
docker run --rm --platform linux/amd64 \
  --mount "type=bind,source=$PWD,target=/work" \
  --workdir /work \
  ghcr.io/fh-campus-wien/hcw-document-builder:1.2.0 \
  render template.qmd
```

Das Ergebnis ist `template.pdf`.

## Schrift

Die originale Helvetica ist proprietär und darf nicht einfach mit dem
öffentlichen Docker-Image verteilt werden. Das Format verwendet deshalb TeX
Gyre Heros. Diese freie OpenType-Schrift basiert auf URW Nimbus Sans und ist
metrisch und optisch mit Helvetica kompatibel. Der HCW-Builder stellt sie für
LuaLaTeX reproduzierbar bereit. Sollte später eine lizenzierte Hochschulschrift
verfügbar sein, kann sie an einer zentralen Stelle im Format ausgetauscht
werden.

## Verwendung in Vorlagen

Quarto-Erweiterungen liegen zur Renderzeit im Projekt unter `_extensions/`.
Eine Dokumentvorlage übernimmt eine freigegebene Version dieses Formats. Damit
bleibt ein geklontes Vorlagenprojekt nach dem einmaligen Download des
Builder-Images auch offline renderbar.

Während der Entwicklung kann das Format direkt aus diesem Repository getestet
werden. Nach der ersten Freigabe wird es versioniert; Dokumentvorlagen
übernehmen dann bewusst eine konkrete Version statt automatisch den jeweils
neuesten Stand.

## Release in eine Dokumentvorlage übernehmen

Quarto kann einen bestimmten GitHub-Tag installieren:

```shell
quarto add FH-Campus-Wien/hcw-quarto-format@v0.1.1 --no-prompt
```

Die installierte Erweiterung liegt anschließend unter
`_extensions/FH-Campus-Wien/hcw/`. Dieser Ordner wird gemeinsam mit der
Dokumentvorlage versioniert. Dadurch ist GitHub nur beim bewussten
Maintainer-Update erforderlich, nicht beim späteren Rendern.

Die gepflegten HCW-Templates kapseln diesen Befehl in einem Docker-basierten
Maintainer-Task und prüfen danach die installierte Versionsnummer sowie alle
erforderlichen Ressourcen.

## Anhang: QMD-, LaTeX- und PDF-Dateien

Beim Rendern entsteht eine Verarbeitungskette:

```text
template.qmd
    ↓ Quarto und Pandoc
template.tex
    ↓ LuaLaTeX
template.pdf
```

Die Dateien haben unterschiedliche Aufgaben:

| Datei | Aufgabe | Versioniert |
|---|---|---|
| `template.qmd` | mit Markdown geschriebenes Testdokument des Formats | ja |
| `template.tex` | automatisch erzeugte LaTeX-Zwischendatei zur Fehlersuche | nein |
| `template.pdf` | gerendertes visuelles Testergebnis | nein |
| `_extensions/hcw/header.tex` | gemeinsame Schrift-, Seiten- und Beschriftungseinstellungen | ja |
| `_extensions/hcw/title.tex` | Aufbau der HCW-Titelseite | ja |

`template.tex` wird erzeugt, weil im Testdokument `keep-tex: true` aktiviert
ist. Die Datei darf gelöscht werden und wird beim nächsten Rendern neu
erstellt. Manuelle Änderungen daran gehen verloren. Dauerhafte
Layoutänderungen gehören ausschließlich in `header.tex`, `title.tex` oder
`_extension.yml`.

Das Logo besitzt ebenfalls nur eine Quelle:

```text
_extensions/hcw/assets/images/hcw-logo-anthrazit.png
```

In einer installierten Dokumentvorlage liegt dieselbe versionierte Datei unter
`_extensions/FH-Campus-Wien/hcw/assets/images/`. Die Titelseite greift direkt
auf diese Extension-Datei zu; eine zusätzliche Logo-Kopie im Projektstamm ist
nicht erforderlich.
