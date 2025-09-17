# AI Legal Advisor Agent - Minimalistische Architectuur
**Project Naam**: AI Legal Advisor Agent (Kleinschalig)
**Versie**: 1.0
**Datum**: 17 september 2025
**Auteurs**: Winston (Architect), Renzo Wouters

---

## 1. Inleiding

### 1.1 Doel van dit Document
Dit document beschrijft een **minimalistische, kleinschalige architectuur** voor de AI Legal Advisor Agent, gericht op **strafrecht** en gebaseerd op **gratis en open-source technologieën**. Het systeem is ontworpen voor **lokaal gebruik** of hosting op een **goedkope VPS** (bijv. €5/mnd), met een focus op **eenvoudige implementatie** en **onderhoudbaarheid**.

### 1.2 Kernprincipes
- **Open Source**: Alle gebruikte technologieën zijn gratis en open source.
- **Lokaal First**: Ontworpen om lokaal te draaien (bijv. op een laptop of Raspberry Pi).
- **Minimalistisch**: Alleen essentiële componenten, geen overbodige complexiteit.
- **Uitbreidbaar**: Modulair ontwerp voor toekomstige uitbreidingen (bijv. andere rechtsgebieden).

---

## 2. High-Level Architectuur

### 2.1 Technische Samenvatting
De AI Legal Advisor Agent is een **lokaal draaiende applicatie** met:
- **Backend**: Python (Flask) voor retrieval en ranking van juridische data.
- **Frontend**: React (Create React App) voor de gebruikersinterface (optioneel; CLI is ook mogelijk).
- **Data Layer**:
  - **SQLite** voor het opslaan van wetten en uitspraken (lokaal, geen server nodig).
  - **FAISS** (Facebook AI Similarity Search) voor semantische zoekopdrachten (lokaal, in-memory of op schijf).
- **Embeddings**: **Sentence Transformers** (`all-MiniLM-L6-v2`) voor gratis, lokale embedding-generatie (geen OpenAI API-kosten).
- **Externe Bronnen**:
  - Handmatig gedownloade XML/JSON-bestanden van [wetten.nl](https://wetten.overheid.nl) en [rechtspraak.nl Open Data](https://data.rechtspraak.nl) (geen live API-calls om kosten/afhankelijkheden te vermijden).

### 2.2 Platform en Infrastructuur
| Categorie          | Technologie               | Versie/Notities                                  |
|--------------------|---------------------------|--------------------------------------------------|
| **Backend**        | Python                    | 3.10+                                            |
| **Web Framework**  | Flask                     | 2.3.2 (lichtgewicht, eenvoudig)                  |
| **Embeddings**     | Sentence Transformers     | `all-MiniLM-L6-v2` (lokaal, gratis)             |
| **Vector DB**      | FAISS                     | 1.7.4 (lokaal, geen server nodig)               |
| **Database**       | SQLite                    | Ingebouwd in Python (geen server)               |
| **Frontend**       | React (Create React App)  | 18.2.0 (optioneel; CLI is voldoende)            |
| **Styling**        | Pure CSS / Tailwind (CDN) | Geen build-step nodig                           |
| **Testing**        | Pytest                    | 7.4.0 (voor backend)                             |
| **CI/CD**          | GitHub Actions (optioneel)| Voor automatische tests (gratis voor open source) |

### 2.3 Repository Structuur

project-root/
├── backend/
│   ├── app.py                # Flask API
│   ├── retrieval.py          # Retrieval-logica
│   ├── ranking.py            # Ranking-logica
│   ├── models.py             # Data-modellen (Wet, Uitspraak)
│   ├── scripts/
│   │   ├── download_wetten.py # Script om wetten te downloaden/parsen
│   │   └── download_uitspraken.py # Script om uitspraken te downloaden
│   ├── data/
│   │   ├── wetten.db         # SQLite database voor wetten
│   │   ├── uitspraken.db     # SQLite database voor uitspraken
│   │   └── faiss_index/      # FAISS-index voor embeddings
│   └── tests/                # Pytest tests
├── frontend/                 # Optioneel: React frontend
│   ├── src/
│   │   ├── components/       # React-componenten
│   │   ├── App.js            # Hoofdcomponent
│   │   └── index.js          # Entry point
│   └── public/               # Statische bestanden
├── docs/                      # Documentatie (PRD, architectuur)
└── README.md                 # Installatie- en gebruiksinstructies


### 2.4 High-Level Architectuurdiagram
```mermaid
graph TD
    A[Gebruiker] -->|Stelt vraag| B[Frontend/CLI]
    B -->|Verstuurd vraag| C[Flask API]
    C -->|Retrieve data| D[FAISS Index]
    C -->|Retrieve data| E[SQLite: Wetten]
    C -->|Retrieve data| F[SQLite: Uitspraken]
    D -->|Retourneert indices| C
    E -->|Retourneert wetten| C
    F -->|Retourneert uitspraken| C
    C -->|Rankt resultaten| G[Ranking Module]
    G -->|Retourneert gerankte resultaten| C
    C -->|Retourneert antwoord| B

3. Tech Stack (Open Source & Gratis)
3.1 Technology Stack Tabel
CategorieTechnologieVersiePurposeRationaleBackend LanguagePython3.10+Primair voor backend-logica.Brede ondersteuning, eenvoudig te installeren.Web FrameworkFlask2.3.2Lichtgewicht API voor lokale ontwikkeling.Geen complexe afhankelijkheden, eenvoudig te deployen.EmbeddingsSentence Transformersall-MiniLM-L6-v2Lokale embedding-generatie (geen API-kosten).Gratis, lokaal draaiend, goede prestaties voor Nederlandse tekst.Vector DBFAISS1.7.4Lokale semantische zoekopdrachten.Geen server nodig, eenvoudig te integreren met Python.DatabaseSQLiteIngebouwdOpslaan van wetten en uitspraken (lokaal, geen server).Geen installatie nodig, werkt out-of-the-box.FrontendReact (Create React App)18.2.0Optionele gebruikersinterface.Eenvoudig op te zetten, goede documentatie.StylingPure CSS / Tailwind (CDN)-Minimale styling zonder build-step.Geen extra tools nodig, werkt met CDN.TestingPytest7.4.0Backend-tests.Standaard voor Python, eenvoudig te gebruiken.CI/CDGitHub Actions-Automatische tests (optioneel).Gratis voor open-source projecten.

4. Data Modellen (Vereenvoudigd)
4.1 Wet Model (SQLite)
 Kopiërenclass Wet:
    id: str          # Bijv. "Sr_123" (Strafrecht, Artikel 123)
    titel: str       # Bijv. ":followup[Wetboek van Strafrecht]{question="Welke specifieke artikelen uit het Wetboek van Strafrecht zijn het meest relevant voor veelvoorkomende juridische vragen?" questionId="54712032-1186-4f1d-bfe5-10e2249d8f6f"}"
    artikel_nummer: str  # Bijv. "123"
    tekst: str       # Volledige tekst van het artikel
    url: str         # URL naar de bron op wetten.nl
    embeddings: str  # Base64-gencodeerde embedding (opslaan als tekst)
    rechtsgebied: str # Bijv. "strafrecht"
4.2 Uitspraak Model (SQLite)
 Kopiërenclass Uitspraak:
    id: str          # ECLI-nummer (bijv. "ECLI\:NL\:RBAMS:2023:123")
    titel: str       # Titel van de uitspraak
    datum: str       # Datum (YYYY-MM-DD)
    rechtbank: str   # Bijv. "Rechtbank Amsterdam"
    tekst: str       # Samenvatting of volledige tekst
    url: str         # URL naar de bron
    embeddings: str  # Base64-gencodeerde embedding
    rechtsgebied: str # Bijv. "strafrecht"
4.3 Database Schema (SQLite)
 Kopiëren-- wetten.db
CREATE TABLE wetten (
    id TEXT PRIMARY KEY,
    titel TEXT NOT NULL,
    artikel_nummer TEXT NOT NULL,
    tekst TEXT NOT NULL,
    url TEXT NOT NULL,
    embeddings TEXT NOT NULL,  -- Base64-encoded embedding
    rechtsgebied TEXT NOT NULL
);

-- uitspraken.db
CREATE TABLE uitspraken (
    id TEXT PRIMARY KEY,
    titel TEXT NOT NULL,
    datum TEXT NOT NULL,
    rechtbank TEXT NOT NULL,
    tekst TEXT NOT NULL,
    url TEXT NOT NULL,
    embeddings TEXT NOT NULL,  -- Base64-encoded embedding
    rechtsgebied TEXT NOT NULL
);

5. Componenten (Minimalistisch)
5.1 Backend: Retrieval Module
Verantwoordelijkheid: Zoeken naar relevante wetten/uitspraken met FAISS.
Afhankelijkheden: FAISS, SQLite, Sentence Transformers.
Voorbeeldcode:
 Kopiërenfrom sentence_transformers import SentenceTransformer
import faiss
import numpy as np
import sqlite3
import base64
import pickle

class RetrievalModule:
    def __init__(self, faiss_index_path: str):
        self.model = SentenceTransformer('all-MiniLM-L6-v2')
        self.index = faiss.read_index(faiss_index_path)

    def search(self, query: str, k: int = 5) -> list:
        # 1. Embed de query
        query_embedding = self.model.encode(query)

        # 2. Zoek in FAISS
        distances, indices = self.index.search(np.array([query_embedding]), k)

        # 3. Haal de wetten/uitspraken op uit SQLite
        conn = sqlite3.connect('data/wetten.db')
        cursor = conn.cursor()
        results = []
        for idx in indices[0]:
            cursor.execute("SELECT * FROM wetten WHERE rowid=?", (idx + 1,))
            wet = cursor.fetchone()
            if wet:
                wet['embeddings'] = pickle.loads(base64.b64decode(wet['embeddings']))
                results.append(wet)
        conn.close()
        return results
5.2 Backend: Ranking Module
Verantwoordelijkheid: Rank resultaten op relevantie (cosine similarity).
Afhankelijkheden: NumPy, SciPy.
Voorbeeldcode:
 Kopiërenfrom scipy.spatial.distance import cosine
import numpy as np

class RankingModule:
    def rank(self, query_embedding: np.ndarray, results: list) -> list:
        # Bereken cosine similarity voor elk resultaat
        ranked = []
        for result in results:
            embedding = pickle.loads(base64.b64decode(result['embeddings']))
            score = 1 - cosine(query_embedding, embedding)
            ranked.append((result, score))

        # Sorteer op score (hoog → laag)
        return sorted(ranked, key=lambda x: x[1], reverse=True)
5.3 Backend: Flask API
Verantwoordelijkheid: Blootstellen van /search endpoint.
Afhankelijkheden: Flask, RetrievalModule, RankingModule.
Voorbeeldcode:
 Kopiërenfrom flask import Flask, request, jsonify

app = Flask(__name__)
retrieval = RetrievalModule("data/faiss_index")
ranking = RankingModule()

@app.route('/api/search', methods=['POST'])
def search():
    data = request.json
    query = data['query']

    # 1. Retrieve resultaten
    results = retrieval.search(query)

    # 2. Rank resultaten
    ranked_results = ranking.rank(retrieval.model.encode(query), results)

    # 3. Retourneer top 5
    return jsonify({
        "results": [{"item": r[0], "score": r[1]} for r in ranked_results[:5]]
    })

if __name__ == '__main__':
    app.run(debug=True)
5.4 Frontend (Optioneel: CLI of React)
CLI-Versie (Minimaal)
Gebruik curl of requests om de Flask API aan te roepen:
 Kopiërencurl -X POST http://localhost:5000/api/search -H "Content-Type: application/json" -d '{"query": "Mag de politie..."}'
React-Versie (Create React App)
Component: SearchField.jsx
 Kopiërenimport { useState } from 'react';

export const SearchField = ({ onSearch }) => {
  const [query, setQuery] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    onSearch(query);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Stel je juridische vraag..."
      />
      <button type="submit">Zoek</button>
    </form>
  );
};
Component: ResultsList.jsx
 Kopiërenexport const ResultsList = ({ results }) => {
  return (
    <div>
      {results.map((result, index) => (
        <div key={index}>
          <h3>{result.item.titel}</h3>
          <p>{result.item.tekst.substring(0, 200)}...</p>
          <a href={result.item.url} target="_blank">Lees meer</a>
          <p>Relevantie: {result.score.toFixed(2)}</p>
        </div>
      ))}
    </div>
  );
};

6. Data Pipeline (Handmatig)
6.1 Wetten Downloaden en Parsen

Handmatig downloaden:

Download XML-bestanden van wetten.nl (bijv. Wetboek van Strafrecht).
Sla op in backend/data/wetten/xml/.


Parsen naar SQLite:
 Kopiëren# backend/scripts/download_wetten.py
import xml.etree.ElementTree as ET
import sqlite3
from sentence_transformers import SentenceTransformer

def parse_wet(xml_file: str):
    tree = ET.parse(xml_file)
    root = tree.getroot()
    wetten = []
    for wet in root.findall('.//wet'):
        wetten.append({
            "id": wet.find("id").text,
            "titel": wet.find("titel").text,
            "artikel_nummer": wet.find("artikel").get("nummer"),
            "tekst": wet.find("tekst").text,
            "url": wet.find("url").text,
            "rechtsgebied": "strafrecht"  # Hardcoded voor MVP
        })
    return wetten

def store_wetten(wetten: list):
    conn = sqlite3.connect('data/wetten.db')
    cursor = conn.cursor()
    cursor.execute("CREATE TABLE IF NOT EXISTS wetten (...)")  # Zie sectie 4.3
    model = SentenceTransformer('all-MiniLM-L6-v2')
    for wet in wetten:
        embedding = model.encode(wet['tekst'])
        wet['embeddings'] = base64.b64encode(pickle.dumps(embedding)).decode('utf-8')
        cursor.execute("INSERT INTO wetten VALUES (?, ?, ?, ?, ?, ?, ?)", (
            wet['id'], wet['titel'], wet['artikel_nummer'],
            wet['tekst'], wet['url'], wet['embeddings'], wet['rechtsgebied']
        ))
    conn.commit()
    conn.close()

if __name__ == "__main__":
    wetten = parse_wet("data/wetten/xml/Sr.xml")
    store_wetten(wetten)


6.2 Uitspraken Downloaden

Handmatig downloaden:

Download JSON-bestanden van rechtspraak.nl Open Data (filter op "strafrecht").
Sla op in backend/data/uitspraken/json/.


Parsen naar SQLite:
 Kopiëren# backend/scripts/download_uitspraken.py
import json
import sqlite3
from sentence_transformers import SentenceTransformer

def parse_uitspraak(json_file: str):
    with open(json_file) as f:
        data = json.load(f)
    uitspraken = []
    for item in data:
        uitspraken.append({
            "id": item["ecli"],
            "titel": item["titel"],
            "datum": item["datum"],
            "rechtbank": item["rechtbank"],
            "tekst": item["tekst"],
            "url": item["url"],
            "rechtsgebied": "strafrecht"
        })
    return uitspraken

def store_uitspraken(uitspraken: list):
    conn = sqlite3.connect('data/uitspraken.db')
    cursor = conn.cursor()
    cursor.execute("CREATE TABLE IF NOT EXISTS uitspraken (...)")  # Zie sectie 4.3
    model = SentenceTransformer('all-MiniLM-L6-v2')
    for uitspraak in uitspraken:
        embedding = model.encode(uitspraak['tekst'])
        uitspraak['embeddings'] = base64.b64encode(pickle.dumps(embedding)).decode('utf-8')
        cursor.execute("INSERT INTO uitspraken VALUES (?, ?, ?, ?, ?, ?, ?, ?)", (
            uitspraak['id'], uitspraak['titel'], uitspraak['datum'],
            uitspraak['rechtbank'], uitspraak['tekst'], uitspraak['url'],
            uitspraak['embeddings'], uitspraak['rechtsgebied']
        ))
    conn.commit()
    conn.close()

if __name__ == "__main__":
    uitspraken = parse_uitspraak("data/uitspraken/json/strafrecht.json")
    store_uitspraken(uitspraken)


6.3 FAISS Index Bouwen
 Kopiëren# backend/scripts/build_faiss_index.py
import faiss
import sqlite3
import numpy as np
import pickle
import base64

def build_index():
    # 1. Laad alle embeddings uit SQLite
    conn = sqlite3.connect('data/wetten.db')
    cursor = conn.cursor()
    cursor.execute("SELECT embeddings FROM wetten")
    wet_embeddings = [pickle.loads(base64.b64decode(row[0])) for row in cursor.fetchall()]

    cursor.execute("SELECT embeddings FROM uitspraken")
    uitspraak_embeddings = [pickle.loads(base64.b64decode(row[0])) for row in cursor.fetchall()]
    conn.close()

    # 2. Combineer embeddings
    all_embeddings = np.array(wet_embeddings + uitspraak_embeddings)

    # 3. Bouw FAISS-index
    index = faiss.IndexFlatL2(all_embeddings.shape[1])
    index.add(all_embeddings)

    # 4. Sla index op
    faiss.write_index(index, "data/faiss_index")

if __name__ == "__main__":
    build_index()

7. Deployment (Lokaal of Goedkope VPS)
7.1 Lokaal Draaien


Backend:
 Kopiërencd backend
pip install -r requirements.txt  # Flask, sentence-transformers, faiss-cpu, etc.
python app.py

Draait op http://localhost:5000.



Frontend (optioneel):
 Kopiërencd frontend
npm install
npm start

Draait op http://localhost:3000.



Data Pipeline:

Voer de scripts uit om data te downloaden en de FAISS-index te bouwen:
 Kopiërenpython scripts/download_wetten.py
python scripts/download_uitspraken.py
python scripts/build_faiss_index.py




7.2 Goedkope VPS (bijv. €5/mnd)

Backend: Draai de Flask-app met gunicorn:
 Kopiërenpip install gunicorn
gunicorn -w 4 -b 0.0.0.0:5000 app\:app

Frontend: Build en serveer de React-app met serve:
 Kopiërennpm run build
npm install -g serve
serve -s build

Data: Kopieer de data/ map naar de VPS (bijv. met scp).
Reverse Proxy: Gebruik Nginx om beide services te exposen:
 Kopiërenserver {
    listen 80;
    server_name jouwdomein.nl;

    location /api/ {
        proxy_pass http://localhost:5000;
    }

    location / {
        proxy_pass http://localhost:3000;
    }
}



8. Testing (Minimaal maar Effectief)
8.1 Backend Tests (Pytest)
Voorbeeld:
 Kopiëren# backend/tests/test_retrieval.py
import pytest
from retrieval import RetrievalModule

def test_retrieval_module():
    # Setup: Bouw een kleine test-index
    retrieval = RetrievalModule("data/test_faiss_index")

    # Act: Zoek met een test-query
    results = retrieval.search("diefstal", k=2)

    # Assert: Controleer of resultaten relevant zijn
    assert len(results) == 2
    assert all("diefstal" in result['tekst'].lower() for result in results)
8.2 Frontend Tests (React Testing Library)
Voorbeeld:
 Kopiëren// frontend/src/SearchField.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import { SearchField } from './SearchField';

test('submits query on form submit', () => {
  const mockOnSearch = jest.fn();
  render(<SearchField onSearch={mockOnSearch} />);

  const input = screen.getByPlaceholderText('Stel je juridische vraag...');
  const button = screen.getByText('Zoek');

  fireEvent.change(input, { target: { value: 'Mag de politie...' } });
  fireEvent.click(button);

  expect(mockOnSearch).toHaveBeenCalledWith('Mag de politie...');
});

9. Kostenanalyse
ComponentTechnologieKostenBackendPython/Flask€0 (open source)EmbeddingsSentence Transformers€0 (lokaal, geen API-kosten)Vector DBFAISS€0 (lokaal)DatabaseSQLite€0 (lokaal)FrontendReact (CRA)€0 (open source)HostingLokaal / VPS€0 (lokaal) of €5/mnd (VPS)CI/CDGitHub Actions€0 (voor open source)Data BronnenHandmatig gedownload€0
Totaal: €0 (lokaal) of €5/mnd (VPS).

10. Next Steps (Minimalistische Implementatie)
10.1 Immediate Actions


Data Verzamelen:

Download handmatig 5-10 relevante wetten (bijv. Wetboek van Strafrecht) en 20-30 uitspraken (strafrecht) van rechtspraak.nl Open Data.
Sla op in backend/data/wetten/xml/ en backend/data/uitspraken/json/.



Backend Setup:

Installeer afhankelijkheden:
 Kopiërenpip install flask sentence-transformers faiss-cpu sqlite3 numpy scipy

Voer de data-pipeline scripts uit om de databases en FAISS-index te bouwen.



Flask API Draaien:

Start de API:
 Kopiërenpython app.py

Test met curl:
 Kopiërencurl -X POST http://localhost:5000/api/search -H "Content-Type: application/json" -d '{"query": "Mag de politie mijn telefoon doorzoeken?"}'




Frontend (Optioneel):

Als je een UI wilt, gebruik Create React App:
 Kopiërennpx create-react-app frontend
cd frontend
npm start

Vervang de placeholder-componenten met SearchField en ResultsList.



10.2 Voorbeeld Vraag/Antwoord
Vraag:
"Mag de politie mijn telefoon doorzoeken zonder toestemming?"
Antwoord (JSON):
 Kopiëren{
  "results": [
    {
      "item": {
        "id": "Sr_123",
        "titel": ":followup[Wetboek van Strafvordering]{question="Welke artikelen uit het Wetboek van Strafvordering zijn het meest relevant voor politieonderzoeken?" questionId="a23681eb-92c0-41f9-81b8-3c6bcfea20a8"}",
        "artikel_nummer": "94",
        "tekst": "De officier van justitie kan bevelen tot :followup[inbeslagname van voorwerpen]{question="Wat zijn de juridische grenzen voor de politie bij het inbeslagnemen van persoonlijke bezittingen, zoals telefoons?" questionId="9f5f060e-7fa5-4d89-9b95-00bb9c7a15f4"} die van belang kunnen zijn voor het onderzoek...",
        "url": "https://wetten.overheid.nl/...",
        "rechtsgebied": "strafrecht"
      },
      "score": 0.92
    },
    {
      "item": {
        "id": "ECLI\:NL\:RBAMS:2023:123",
        "titel": "Uitspraak over telefoononderzoek",
        "datum": "2023-01-12",
        "rechtbank": "Rechtbank Amsterdam",
        "tekst": "In deze zaak oordeelde de rechtbank dat het doorzoeken van een telefoon zonder toestemming gerechtvaardigd was omdat er een :followup[redelijk vermoeden]{question="Hoe wordt 'redelijk vermoeden' juridisch gedefinieerd in Nederlandse wetgeving?" questionId="e3bb6f32-86a1-462b-80c9-95cf202e7f2f"} bestond van betrokkenheid bij een misdrijf...",
        "url": "https://data.rechtspraak.nl/...",
        "rechtsgebied": "strafrecht"
      },
      "score": 0.88
    }
  ]
}

11. Uitbreidingsmogelijkheden

Meer Rechtsgebieden:

Voeg nieuwe scripts toe voor andere rechtsgebieden (bijv. download_bestuurlijk_recht.py).
Breid de FAISS-index uit met nieuwe embeddings.


Betere Ranking:

Voeg citatie-analyse toe (bijv. "Hoe vaak wordt dit artikel geciteerd in uitspraken?").
Gebruik hybride zoekopdrachten (keyword + semantisch).


Automatische Updates:

Schrijf een script dat wekelijks nieuwe uitspraken downloadt en de index bijwerkt.


CLI-Tool:

Bouw een command-line interface voor gebruik zonder frontend:
 Kopiëren# cli.py
import requests

def search_cli():
    query = input("Stel je vraag: ")
    response = requests.post(
        "http://localhost:5000/api/search",
        json={"query": query}
    ).json()
    for result in response["results"]:
        print(f"--- {result['item']['titel']} (score: {result['score']:.2f})")
        print(result['item']['tekst'][:200] + "...")
        print(f"Bron: {result['item']['url']}\n")

if __name__ == "__main__":
    search_cli()
