Gruppenmitglieder:
Calvin Vi 582405
Felicitas Faupel 589495
Okan Inci 589244


Projekt:Buchgenre-Klassifikation

Business Understanding
Ziel des Projekts ist es, automatisch Bücher aus öffentlichen Datenquellen (OpenLibrary und Google Books API) zu sammeln und diese basierend auf Titel und Beschreibung in passende Genres zu klassifizieren.
Dies soll helfen, große Buchmengen systematisch zu kategorisieren, insbesondere wenn keine explizite Genreangabe vorhanden ist.
Das System kombiniert vorhandene Genre-Metadaten (falls verfügbar) mit einer inhaltlichen Textanalyse, um Bücher Genres wie „Romance“, „Science Fiction“, „Fantasy“, „Crime“, „History“, „Horror“ oder „Cooking“ zuzuordnen.Da eine manuelle Klassifikation bei Tausenden von Büchern nicht praktikabel ist, ist ein automatisierter, robust trainierter Klassifikator notwendig.
Das Ziel ist es, ein robustes Klassifikationsmodell zu entwickeln, das mindestens 75–80 % Genauigkeit erreicht.



Data Understanding:

Die Daten wurden aus der OpenLibrary API und der Google Books API extrahiert.
OpenLibrary API liefert Metadaten zu Büchern#(Titel, Autoren, Veröffentlichungsjahr).
Google Books API ergänzt die Beschreibungen der Bücher (soweit verfügbar).
Es wird ein gemeinsamer Datensatz im CSV-Format erstellt.Er enthält title,authors,published,genre,description.


Datensatz-Struktur:
In den Rohdaten books_data.csv enthält jeder Eintrag title,authors,published,genre,description.

Die "gesäuberte" Daten enthalten zusätzlich noch text (Kombination aus Titel und Beschreibung) und label (numerisch kodiertes Genre).

Quantität & Deckung:
Insgesamt wurden ca. 14 000 Roh-Datensätze über sieben Genres („fantasy“, „romance“, „science_fiction“, „crime“, „history“, „horror“
,cooking“) gesammelt.

Nach Filterung fehlender oder nicht‑englischer Beschreibungen blieben ca. 6 535 Einträge übrig.


Data Preparation:

Textbereinigung: Entfernen von HTML-Tags, Sonderzeichen und Umwandeln der Texte in Kleinbuchstaben.

Bereinigung & Sprachfilterung:
Entfernen aller Einträge ohne englische Beschreibung ("No description") oder mit fehlender Beschreibung (NaN).
Einsatz der Bibliothek langdetect, um sicherzustellen, dass nur englischsprachige Texte im Datensatz verbleiben.
Mit einer kleinen Hilfsfunktion (clean_description) wurde in Fällen, in denen die Beschreibung mit dem Titel beginnt, dieser vorn abgeschnitten, um Redundanzen zu vermeiden.
Feature-Erstellung: Titel und Beschreibung wurden zu einem „text“-Feld zusammengefügt.

TF-IDF Vektorisierung: Als Feature-Extraktion für klassische ML-Modelle wurde ein TF-IDF-Vektorraum mit bis zu 10.000 Merkmalen (1-2-Gramme) genutzt.

Label Encoding: Die Ziel-Genres wurden mit dem LabelEncoder in numerische Labels umgewandelt.

Oversampling: Ein RandomOverSampler wurde verwendet, um das Ungleichgewicht der Klassen im Training auszugleichen.



Modelling:

Zunächst haben wir verschiedene klassische Machine-Learning-Modelle (Logistische Regression, Multinomial Naive Bayes, Linear SVC, Random Forest) auf einem TF‑IDF–Vektorraum trainiert, um einen schnellen Benchmark zu erhalten.

Um mit ungleich verteilten Klassen zurechtzukommen, wendeten wir im Training ein RandomOverSampling an.

Als fortgeschrittenes Modell setzten wir schließlich auf DistilBERT („distilbert‑base‑uncased“) aus der Huggingface‑Bibliothek.

Wir tokenisierten die bereinigten Texte (Titel + Beschreibung) mit dem DistilBERT‑Tokenizer (maximale Länge 512).

In einem Hyperparameter‑Loop testeten wir verschiedene Lernraten (2 × 10⁻⁵, 5 × 10⁻⁵) und Batch‑Größen (8, 16) über jeweils 5 Epochen, mit Evaluation nach jeder Epoche und Early‑Stopping (Patience = 2).

Das am besten generalisierende Setup (lr=2e‑5, bs=8) wurde als finales Modell ausgewählt.


Evaluation:

Metriken: Wir betrachteten Accuracy, Precision, Recall und F1‑Score (Macro und Weighted).

Baseline: Ein DummyClassifier lieferte eine Accuracy von ca. 20 %.

Klassische Modelle: Die Logistische Regression erreichte etwa 77 % Accuracy und Macro‑F1 ≈ 0.75.

DistilBERT: Auf dem Test‑Split erzielte unser bestes BERT‑Modell 83 % Accuracy und einen Macro‑F1 von ≈ 0.82.

Lernkurven: Die Trainings‑Loss sank stetig, während die Validation‑Loss nach anfänglichem Rückgang leicht wieder anstieg – ein Zeichen, dass 5 Epochen ausreichend waren, um Überanpassung zu vermeiden.













Kurzbeschreibung der wichtigsten Files
data/books_data.csv: Rohdaten nach dem Scraping aller Genres; enthält Titel, Autoren, Jahr, Genre, Beschreibung.  
data/clean_books.csv: Finaler Datensatz fürs Modell: nur englische Beschreibungen, Duplikate & “No description”‑Einträge entfernt, mit kombinierten `text`- und `label`-Spalten.  

bookgenreprediction.ipynbHolt daten von openlibrary und google.säubert diese,trainiert und vergleicht modelle wie Logistic Regression,gibt Evaluation Accuracy, Precision/Recall/F1
bert-prediction.ipynb:trainiert das DistilBERT-Modell,zeigt auch die loss curve

model.pkl :Gespeichertes LogisticRegression‑Modell
tfidf.pkl : Gespeicherter TF‑IDF‑Vektorisierer
label_encoder.pkl : Gespeicherter LabelEncoder
best_bert_model: Ordner mit dem feingetunten DistilBERT‑Modell und den dazugehörigen Tokenizer‑Dateien.  

requirements.txt: Liste aller benötigten Bibliotheken und Versionsvorgaben.  
README.md: Diese Datei – enthält Projekt­ziele, Ordner­struktur, How‑to‑Run und Kontaktinfos.

