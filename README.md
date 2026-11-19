# 🌍 GEO-Optimierung bestehender Blogartikel mit Python und LLM

> Eine modulare Python-Pipeline, die ältere Blogartikel datenbasiert für KI-Suchmaschinen aufbereitet (Generative Engine Optimization, GEO): von der Analyse in der Google Search Console bis zur Aktualisierung in WordPress.

---

## Rolle & Entwicklungsansatz

* **Rolle:** Christoph Bauer, Business Analyst: Problemanalyse, Auswahllogik der Artikel, Datenfluss zwischen Search Console, LLM und WordPress, Schutzmechanismen, Prompt-Design und Test.
* **Umsetzung:** Den Python-Code habe ich mit KI-Coding-Assistenten erstellt und anschließend getestet und angepasst.

---

## Ausgangslage und Lösung

* **Ausgangslage:** Ein älterer Blog hatte über längere Zeit stabile Impressionen, aber eine niedrige Klickrate (CTR), unter anderem wegen fehlender Struktur und fehlender Signale für moderne Such- und KI-Systeme. Rund 90 Artikel von Hand zu überarbeiten war zeitlich nicht machbar.
* **Lösung:** Eine Pipeline, die Artikel mit Potenzial aus der Google Search Console ermittelt, sie per LLM um strukturierte Inhalte ergänzt und über die WordPress REST API aktualisiert.
* **Ergebnis:** Ausgewählte Artikel werden automatisch um FAQs, Infoboxen, Tabellen und strukturierte Daten ergänzt. Schutzmechanismen verhindern doppelte Einfügungen und beschädigte Inhaltsverzeichnisse.

---

## Tech Stack

* **Daten & Automatisierung:** Python (`pandas`, `requests`, `BeautifulSoup`), Google Search Console API
* **LLM:** OpenAI API (`gpt-4o` mit JSON-Ausgabe; Prompt-Vorgabe, nur Fakten aus dem Artikel zu verwenden)
* **CMS:** WordPress REST API (HTML-Blöcke für den Gutenberg-Editor, Yoast-SEO-Metadaten)
* **Zugänge:** Search Console über ein Google-Service-Konto (`client_secret.json`, nicht im Repository); WordPress-Zugangsdaten und API-Schlüssel sind im Code durch Platzhalter (`"----"`) ersetzt

### Wichtige Designentscheidungen

1. **Abrufe mit Browser-Kennung:** Ein realistischer User-Agent verhindert, dass die eigenen Seiten beim Auslesen von Firewall-Regeln blockiert werden.
2. **Gekapselte HTML-Blöcke:** Neues HTML wird in eigene Gutenberg-Blöcke (`<!-- wp:html -->`) eingefügt, damit Editor und Inhaltsverzeichnis intakt bleiben.
3. **Keine doppelten Einfügungen:** Eine Markierung (`📊 Fakten & Übersicht`) zeigt an, dass ein Artikel schon bearbeitet wurde. Wiederholte Läufe überspringen ihn.

---

## Aufbau

Die Module lassen sich einzeln oder gesammelt über den Batch-Runner starten:

1. **Search-Console-Analyse (`gsc_audit.py`):** Lädt Leistungsdaten über die API und filtert Artikel mit Potenzial (viele Impressionen, niedrige CTR, Position ≤ 20).
2. **Inhaltsaufbereitung (`geo_transformer.py`):** Liest den Artikel per BeautifulSoup aus, unterscheidet nach Artikeltyp (z. B. Reise oder Groundhopping) und erzeugt per LLM FAQs, Infoboxen und Tabellen.
3. **WordPress-Aktualisierung (`batch_geo_optimizer.py` / `run_geo_optimizer.py`):** Ermittelt die Post-ID über den URL-Slug, prüft auf frühere Bearbeitung und schreibt die Ergänzungen in den Artikel.

---

## Umsetzung im Detail

### A. Auswahl der Artikel (`gsc_audit.py`)
* Zugriff auf die Search Console API über ein Service-Konto.
* Export der Kandidaten als `gsc_quick_wins.csv`.

### B. Inhaltsergänzung per LLM (`geo_transformer.py`)
* Der Prompt erlaubt nur Fakten aus dem jeweiligen Artikeltext, um erfundene Angaben zu vermeiden.
* Erzeugt Yoast-Metadaten, Kurzinfo-Boxen und strukturierte Daten nach Schema.org (`FAQPage`, JSON-LD).

### C. Veröffentlichung in WordPress
* Post-ID-Ermittlung über die REST API (`/wp/v2/posts?slug=...`).
* Responsive HTML-Tabellen im Design des Blogs; liefert das LLM keine verwertbare Antwort, wird der Artikel nicht verändert.

---

## Ausführung

![Script Execution Log](Pictures/script-execution-log.png)

---

## Learnings

1. **Struktur vor Keywords:** Für KI-gestützte Suche zählt, dass Informationen maschinenlesbar aufbereitet sind (FAQ-Schema, kompakte Tabellen).
2. **Vorsicht bei Live-Systemen:** Wer automatisiert in ein CMS schreibt, braucht feste Schutzmechanismen gegen doppelte oder fehlerhafte Inhalte.
3. **Klare Vorgaben für die KI:** Gute Ergebnisse entstehen durch präzise Prompts, klare Architekturvorgaben und Kontrollschritte.

---

## Repository-Struktur

```text
├── Scripts/
│   ├── batch_geo_optimizer.py   # Batch-Runner für mehrere Artikel
│   ├── geo_transformer.py       # Inhaltsaufbereitung per LLM
│   ├── gsc_audit.py             # Auswahl über die Search Console API
│   ├── run_geo_optimizer.py     # Testlauf für einen einzelnen Artikel
│   └── wp_publisher.py          # Gezielte Einzel-Updates
├── Docs/
│   └── gsc_quick_wins.csv       # Ergebnis der Potenzialanalyse
├── Pictures/
│   └── script-execution-log.png
└── README.md
```
