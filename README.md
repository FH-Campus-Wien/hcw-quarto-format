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
verfügbar sein, kann sie über den folgenden kontrollierten Prozess ausgetauscht
werden.

### Schrift austauschen

Die Schriftdateien und die Schriftauswahl haben getrennte Zuständigkeiten:

| Repository | Zuständigkeit |
|---|---|
| `hcw-document-builder` | stellt die Schrift im Docker-Image bereit |
| `hcw-quarto-format` | wählt die Schriftfamilien für Fließtext, Überschriften und Code |
| `hcw-template-*` | übernimmt freigegebene Builder- und Formatversionen |

Der Austausch wird von den Maintainer:innen in dieser Reihenfolge durchgeführt:

1. **Lizenz prüfen.** Die Lizenz muss die Verteilung im öffentlichen
   GitHub-Repository und im öffentlichen Docker-Image ausdrücklich erlauben.
   Ist das nicht erlaubt, darf die Schrift dort nicht eingebunden werden. In
   diesem Fall bleibt TeX Gyre Heros aktiv oder es wird eine getrennte,
   nichtöffentliche Toolchain benötigt.
2. **Schrift im Builder bereitstellen.** Eine frei paketierte Schrift wird in
   `hcw-document-builder/Dockerfile` als Betriebssystempaket ergänzt. Eigene
   `.otf`- oder `.ttf`-Dateien dürfen nur bei entsprechender Lizenz in das Image
   kopiert und im Container registriert werden.
3. **Smoke-Test umstellen.** In
   `hcw-document-builder/tests/preamble.tex` werden `\setmainfont` und
   `\setsansfont` auf den exakten internen Familiennamen geändert. Der
   Docker-Build muss diesen PDF-Test erfolgreich abschließen; damit ist
   nachgewiesen, dass LuaLaTeX die Schrift tatsächlich findet.
4. **Format konfigurieren.** In `_extensions/hcw/_extension.yml` werden die
   zentralen Fontoptionen angepasst:

   ```yaml
   mainfont: Name der Schrift
   sansfont: Name der Schrift
   monofont: Name der Codeschrift
   ```

   `mainfont` bestimmt den Fließtext, `sansfont` insbesondere Überschriften und
   `monofont` Codeblöcke. Der interne Familienname muss exakt mit dem im
   Container installierten Namen übereinstimmen.
5. **Lokal gemeinsam testen.** Zuerst wird das geänderte Builder-Image lokal
   gebaut:

   ```bash
   cd hcw-document-builder
   docker build \
     --platform linux/amd64 \
     --build-arg BUILD_VERSION=font-test \
     --tag hcw-document-builder:font-test \
     .
   ```

   Anschließend wird im Format-Repository mit genau diesem Image gerendert:

   ```bash
   cd ../hcw-quarto-format
   docker run --rm --network none --platform linux/amd64 \
     --mount "type=bind,source=$PWD,target=/work" \
     --workdir /work \
     hcw-document-builder:font-test \
     render template.qmd
   ```

   `template.pdf` wird visuell kontrolliert. Zu prüfen sind insbesondere
   Titelseite, Umlaute, fett und kursiv gesetzter Text, Überschriften,
   Tabellen, Formeln, Bildunterschriften und Code.
6. **Builder veröffentlichen.** Erst nach erfolgreichem Test wird eine neue
   Builder-Version getaggt und das öffentliche Image veröffentlicht, zum
   Beispiel `hcw-document-builder:1.3.0`.
7. **Format veröffentlichen.** Danach wird die Builder-Version in den
   Testbefehlen dieses Repositories aktualisiert, die Formatversion in
   `_extension.yml` erhöht und ein neuer Format-Tag veröffentlicht.
8. **Dokumentvorlagen aktualisieren.** In jeder Vorlage werden der
   Builder-Image-Tag, `hcw-format-version.txt`, die VS-Code-Tasks und die
   dokumentierten Docker-Befehle angepasst. Anschließend wird der Task
   `Maintainer: HCW-Format aktualisieren` ausgeführt.
9. **Offline-Abnahme durchführen.** Jede betroffene Vorlage wird abschließend
   mit `--network none` gerendert und die erzeugte PDF visuell geprüft. Erst
   Builder-Version, Formatkopie und Versionsdatei gemeinsam committen.

Eine reine Änderung von `mainfont` ohne Bereitstellung derselben Schrift im
Builder ist nicht ausreichend: Sie würde erst beim PDF-Build mit einer
fehlenden Schrift scheitern. Umgekehrt verändert eine zusätzlich installierte
Schrift das Layout nicht, solange das Format sie nicht auswählt.

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
