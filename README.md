# DHBW Neue Konzepte – Portfolio

## Setup

Dieses Portfolio verwendet [uv](https://docs.astral.sh/uv/) als Python-Paketmanager. `uv` ist ein schneller Ersatz für `pip` und `venv` und verwaltet automatisch die virtuelle Umgebung sowie alle Abhängigkeiten aus der `pyproject.toml`.

### Voraussetzungen

`uv` installieren (einmalig):

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Abhängigkeiten installieren

```bash
uv sync
```

Das erstellt automatisch eine virtuelle Umgebung (`.venv`) und installiert alle Pakete.

---

## Jupyter Notebooks ausführen

### Option 1: VS Code

1. Erweiterung **Jupyter** in VS Code installieren
2. Eine `.ipynb`-Datei aus dem Ordner `notebooks/` öffnen
3. Oben rechts auf **"Select Kernel"** klicken → **"Python Environments"** → `.venv` auswählen

VS Code erkennt die lokale `.venv` in der Regel automatisch.

### Option 2: Jupyter Server im Browser

```bash
uv run jupyter notebook
```

Der Browser öffnet sich automatisch. Alternativ die angezeigte URL manuell aufrufen.

---

## Datensätze

Beide Datensätze stammen von Kaggle und enthalten kurze Texte mit zugeordneten Emotionslabels. Nach dem Parsing haben sie ein einheitliches Schema und identische Emotionskategorien – was eine spätere Zusammenführung ermöglicht.

| Datensatz | Quelle | Zeilen | ID-Spalte | Inhalt |
| --- | --- | --- | --- | --- |
| `tweet_emotion_dataset.csv` | [Kaggle – pashupatigupta](https://www.kaggle.com/datasets/pashupatigupta/emotion-detection-from-text) | 40 000 | `tweet_id` | Tweets |
| `emotion_sentiment_dataset.csv` | [Kaggle – simaanjali](https://www.kaggle.com/datasets/simaanjali/emotion-analysis-based-on-text) | 839 555 | `text_id` | Allgemeine Texte |

Emotionskategorien ($n=13$): `anger`, `boredom`, `empty`, `enthusiasm`, `fun`, `happiness`, `hate`, `love`, `neutral`, `relief`, `sadness`, `surprise`, `worry`

### Parsing-Logik (`notebooks/0_load-and-parse-datasets.ipynb`)

Die Rohdaten werden mit [Polars](https://pola.rs/) eingelesen. Jeder Datensatz durchläuft dieselbe Transformation:

1. **Umbenennen** – Spaltenbezeichnungen aus den Rohdaten auf das einheitliche Schema mappen (`sentiment → emotion`, `text → content` etc.)
2. **Projektion** – nur `content` und `emotion` behalten, restliche Spalten verwerfen
3. **ID hinzufügen** – synthetischen Zeilenindex (`tweet_id` / `text_id`) als `u32` voranstellen
4. **Kategoriencheck** – sicherstellen, dass beide Datensätze exakt dieselben Emotionslabels enthalten (`DataFrame.equals` auf den sortierten Unique-Werten)
5. **Export** – als Parquet speichern: effiziente Spaltenkompression und deutlich schnelleres Laden in Folge-Notebooks als CSV
