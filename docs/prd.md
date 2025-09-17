# PRD content with improvements, split into smaller parts for clarity
prd_content_part1 = """# Product Requirements Document (PRD): AI Legal Advisor Agent

**Project Naam**: AI Legal Advisor Agent
**Versie**: 1.1
**Datum**: 17 september 2025
**Auteurs**: Renzo Wouters, BMAD Analyst Team

---

## 1. Goals and Background Context

### 1.1 Doel

Bouw een **AI Legal Advisor Agent** die juridisch advies kan geven op basis van **Nederlandse wetgeving en jurisprudentie**, met een **MVP-focus op strafrecht**. Het systeem moet later modulair uitbreidbaar zijn naar andere rechtsgebieden (bijv. bestuurlijk recht, civiel recht).

**Roadmap voor uitbreiding**:
1. **Fase 1 (MVP)**: Strafrecht (3 maanden).
2. **Fase 2**: Bestuursrecht (6 maanden na MVP).
3. **Fase 3**: Civiel recht (12 maanden na MVP).

### 1.2 Probleemstelling

- Juridische informatie is versnipperd over **wetten.nl** (wetgeving) en **rechtspraak.nl** (jurisprudentie).
- Het is moeilijk voor niet-juristen (bijv. burgers, kleine organisaties) om te bepalen of hun handelen **binnen de wet** valt.
- Handmatig zoeken is tijdrovend en foutgevoelig, vooral als **wetgeving en jurisprudentie** beide moeten worden geanalyseerd.

**Concreet voorbeeld**:
*Een burger wil weten of de politie zijn telefoon mag doorzoeken zonder toestemming. Het systeem moet artikel 94 van het Wetboek van Strafvordering tonen + relevante uitspraken van de Rechtbank Amsterdam over telefoononderzoek.*

### 1.3 Background Context

- **Wetten.nl** biedt **XML-bestanden** van alle Nederlandse wetten (bijv. Wetboek van Strafrecht).
- **Rechtspraak.nl** biedt een **Open Data API** voor uitspraken (bijv. vonnissen in strafzaken).
- **Vector databases** (FAISS, Weaviate) en **embeddings** (OpenAI, Nomic, Sentence Transformers) maken **semantische zoekopdrachten** mogelijk.

**Alternatieven voor kritieke componenten**:
- **FAISS**: Weaviate of Pinecone voor betere schaalbaarheid.
- **Sentence Transformers**: `paraphrase-multilingual-MiniLM-L12-v2` voor betere meertalige ondersteuning.

---

## 2. Requirements

### 2.1 Functional Requirements (FR)

| ID  | Beschrijving | Prioriteit | Concreet Voorbeeld |
|-----|-------------|------------|------------------------|
| FR1 | De agent moet **wetten en artikelen** kunnen opzoeken op basis van een natuurlijke-taalvraag. | Hoog | *Wat zegt artikel 310 van het Wetboek van Strafrecht over diefstal?* → Toont artikel 310 + uitleg. |
| FR2 | De agent moet **relevante jurisprudentie** (uitspraken) kunnen vinden en koppelen aan wetten. | Hoog | *Zijn er recente uitspraken over diefstal met geweld?* → Toont 3 recente vonnissen + wetten. |
| FR3 | De agent moet een **juridisch onderbouwd antwoord** geven met verwijzingen naar bronnen. | Hoog | *Mag de politie mijn telefoon doorzoeken?* → Toont artikel 94 WvSv + uitspraak ECLI:NL:RBAMS:2023:123. |
| FR4 | De agent moet **uitleggen waarom** een bepaald artikel of uitspraak relevant is. | Hoog | *Dit artikel is relevant omdat het gaat over inbeslagname van bewijsmateriaal.* |
| FR5 | De agent moet **actuele wetgeving en jurisprudentie** gebruiken (automatische updates). | Middel | Wekelijkse updates via cron job. |
| FR7 | De agent moet kunnen uitleggen wat de **juridische consequenties** zijn van een handeling. | Hoog | *Wat zijn de gevolgen als ik een boete niet betaal?* → Toont boeteregeling + mogelijke vervolgstappen. |
| FR6 | Het systeem moet **uitbreidbaar** zijn naar andere rechtsgebieden. | Laag | Modulair ontwerp voor nieuwe domeinen. |

### 2.2 Non-Functional Requirements (NFR)

| ID  | Beschrijving | Metriek | Meetmethode |
|-----|-------------|---------|-----------------|
| NFR1 | Antwoorden moeten binnen **5 seconden** gegenereerd worden. | Responstijd < 5s | Gemeten met `time curl` of Postman. |
| NFR2 | Nauwkeurigheid van antwoorden moet **minimaal 90%** zijn. | 90% correct | Testset van 50 juridische vragen, beoordeeld door een expert. |
| NFR3 | Het systeem moet **schaalbaar** zijn voor 100+ gelijktijdige gebruikers. | 100+ requests/sec | Load testing met Locust of JMeter. |
| NFR4 | Alle antwoorden moeten **traceerbaar** zijn naar bronnen. | 100% bronvermelding | Handmatige controle van 20 willekeurige antwoorden. |
| NFR5 | Het systeem moet **modulair** zijn. | < 1 dag werk per nieuw domein | Tijdmeting bij toevoegen van nieuw rechtsgebied. |

---

## 3. User Interface Design Goals

### 3.1 Target Users

- **Primair**: Juridische professionals (advocaten, officieren van justitie) en **overheidsmedewerkers** (handhavers, beleidsmedewerkers).
- **Secundair**: Burgers die juridisch advies zoeken over **strafrechtelijke kwesties** (boetes, klachten).

**Toegankelijkheid**:
- WCAG AA-compliant (voldoende contrast, toetsenbordnavigatie).
- Dark mode voor betere leesbaarheid.
- Eenvoudige taal voor niet-juridische gebruikers (uitleg van termen).

### 3.2 Core Screens

1. **Startscherm**:
   - Veld voor juridische vraag (bijv. *Mag de politie mijn telefoon doorzoeken zonder toestemming?*).
   - Dropdown voor rechtsgebied (MVP: alleen "strafrecht").
   - Wireframe:
     ```
     +-----------------------------------+
     | [Logo] AI Legal Advisor           |
     |                                   |
     | [ Vraag invoeren...           ]  |
     | [ Rechtsgebied: Strafrecht ▼   ]  |
     | [ Zoek ]                          |
     +-----------------------------------+
     ```

2. **Resultatenscherm**:
   - Lijst met **relevante wetten en uitspraken**, gerangschikt op relevantie.
   - Filteropties (bijv. "Alleen wetten", "Alleen uitspraken").
   - Wireframe:
     ```
     +-----------------------------------+
     | Resultaten (5)                   |
     | [▼ Sorteer op: Relevantie]        |
     |                                   |
     | 1. Wetboek van Strafrecht - Art. 94|
     |    "Inbeslagname van voorwerpen..."
     |    [Lees meer] [Bron: wetten.nl]  |
     |                                   |
     | 2. Uitspraak: ECLI:NL:RBAMS:2023:123 |
     |    "Rechtbank oordeelt dat..."     |
     |    [Lees meer] [Bron: rechtspraak.nl] |
     +-----------------------------------+
     ```

3. **Detailscherm**:
   - Volledige tekst van wet/uitspraak + **uitleg van de agent**.
   - **Bronvermelding** (URL naar wetten.nl/rechtspraak.nl).

### 3.3 Accessibility

- Toetsenbordnavigatie: Tab-index voor alle interactieve elementen.
- Schermlezercompatibiliteit: ARIA-labels voor knoppen en links.
- Dark mode: Optie in instellingen.

---

## 4. Technical Assumptions

### 4.1 Tech Stack

| Categorie       | Technologie               | Versie/Notities                          | Alternatieven |
|-----------------|---------------------------|------------------------------------------|----------------------|
| Backend         | Python                    | 3.10+                                    | -                    |
| Embeddings      | Sentence Transformers     | `all-MiniLM-L6-v2` (lokaal, gratis)       | `paraphrase-multilingual-MiniLM-L12-v2` |
| Vector DB        | FAISS                     | 1.7.4 (lokaal)                           | Weaviate, Pinecone     |
| APIs             | wetten.nl, rechtspraak.nl | Officiële bronnen                       | -                    |
| Frontend         | React/Next.js             | Voor de gebruikersinterface              | Vue.js, Svelte        |
| Hosting          | AWS/GCP                   | Voor schaalbaarheid                      | Azure, lokaal         |
| CI/CD            | GitHub Actions            | Automatische tests                       | GitLab CI, Jenkins    |

### 4.2 Repository Structure

```
project-root/
├── backend/
│   ├── scripts/          # Scripts voor downloaden/parsen
│   ├── retrieval/        # Logica voor semantische zoekopdrachten
│   ├── ranking/          # Ranking-logica
│   ├── api/              # API-endpoints
│   └── data/             # Wetten, uitspraken, FAISS-index
├── frontend/             # React/Next.js
│   ├── components/       # Zoekveld, resultatenlijst
│   ├── pages/            # Startscherm, detailscherm
│   └── styles/           # CSS/Tailwind
├── docs/                 # PRD, architectuur
└── README.md             # Installatie, voorbeelden
```

### 4.3 Coding Standards

- **Python**: PEP 8 + type hints.
- **Frontend**: TypeScript + ESLint.
- **Testing**:
  - Unit tests voor backend (Pytest).
  - E2E tests voor frontend (Cypress).
- **Documentatie**: Docstrings voor alle functies; README met installatie- en gebruiksinstructies.

---

## 5. Epic and Story Structure

### 5.1 Epic 1: Corpus en Retrieval Systeem

**Doel**: Bouw een systeem om wetten en uitspraken te downloaden, parsens, en indexeren.

| Story ID | Beschrijving | Acceptatiecriteria |
|----------|--------------|--------------------|
| S1.1     | Script om wetten van wetten.nl te downloaden/parsen. | XML → SQLite met artikelen + metadata. |
| S1.2     | Script om uitspraken van rechtspraak.nl op te halen. | JSON → SQLite met ECLI + metadata. |
| S1.3     | Vector database met embeddings (FAISS). | FAISS-index met wetten + uitspraken. |
| S1.4     | Back-upmechanisme voor SQLite en FAISS-indexen. | Dagelijkse back-ups naar `/backups/`. |

### 5.2 Epic 2: Agent en Ranking Logica

**Doel**: Bouw de AI-agent die vragen beantwoordt met geïndexeerde data.

| Story ID | Beschrijving | Acceptatiecriteria |
|----------|--------------|--------------------|
| S2.1     | Natuurlijke-taalvraag invoerveld in frontend. | Vraag → backend API. |
| S2.2     | Retrieval van wetten/uitspraken + ranking. | Top-5 resultaten met scores. |
| S2.3     | Uitleg waarom een artikel/uitspraak relevant is. | Antwoord bevat samenvatting + uitleg. |
| S2.4     | Uitleggen juridische consequenties van een handeling. | Antwoord bevat mogelijke consequenties (bijv. boetes, vervolging). |

### 5.3 Epic 3: Frontend en Gebruikersinterface

**Doel**: Bouw een gebruiksvriendelijke interface.

| Story ID | Beschrijving | Acceptatiecriteria |
|----------|--------------|--------------------|
| S3.1     | Eenvoudig invoerveld voor vragen. | Dropdown voor rechtsgebied. |
| S3.2     | Resultatenlijst met filters. | Sorteer op relevantie/datum. |
| S3.3     | Detailscherm met uitleg + bronnen. | Volledige tekst + URL. |
| S3.4     | Dark mode voor betere leesbaarheid. | Toggle voor dark/light mode in instellingen. |

### 5.4 Epic 4: Uitbreiding naar Andere Rechtsgebieden

**Doel**: Modulair ontwerp voor nieuwe domeinen.

| Story ID | Beschrijving | Acceptatiecriteria |
|----------|--------------|--------------------|
| S4.1     | Modulair corpus-ontwerp. | Nieuwe domeinen in `/data/wetten/<domein>/`. |
| S4.2     | Retrieval-logica voor meerdere domeinen. | Parameter `rechtsgebied` in API. |
| S4.3     | Plugin-systeem voor nieuwe zoekalgoritmen. | Nieuwe algoritmen als plugin toevoegen. |

---

## 6. Checklist Results

- **Functionele Eisen**: Alle FR's gedekt door epics/stories.
- **Technische Eisen**: Tech stack voldoet aan NFR1-NFR5.
- **Risico's**:
  - **Juridische aansprakelijkheid**: Gemitigeerd door disclaimers en bronvermelding.
  - **Data-actualiteit**: Opgelost door wekelijkse updates.
  - **Schaalbaarheid**: FAISS/Weaviate + cloud-hosting.

---

## 7. Next Steps

1. **Architectuurdocument bijwerken** met:
   - Back-upstrategieën voor data.
   - Monitoring (Prometheus + Grafana).
   - Unit/integration tests.
2. **Prototype bouwen** voor:
   - Retrieval + ranking (Epic 1 + 2).
   - Frontend (Epic 3).
3. **Testen** met 50 strafrechtelijke vragen.
4. **Stakeholder Review** door juridische experts en ontwikkelaars.
```

prd_content_part2 = """
---

## Appendix: Additional Notes

### A. Data Sources

- **Wetten.nl**: XML feeds van Nederlandse wetgeving.
- **Rechtspraak.nl**: Open Data API voor jurisprudentie.
- **Vector Embeddings**: Sentence Transformers voor semantische zoekopdrachten.

### B. Testing Strategy

- **Unit Tests**: Pytest voor backend functionaliteit.
- **Integration Tests**: Testen van API endpoints en database interacties.
- **E2E Tests**: Cypress voor frontend workflows.
- **Load Tests**: Locust of JMeter voor schaalbaarheidstesten.

### C. Deployment Plan

- **Staging Environment**: Voor interne tests.
- **Production Environment**: Voor live gebruik.
- **CI/CD Pipeline**: GitHub Actions voor automatische deployments.

### D. Risk Mitigation

- **Data Accuracy**: Wekelijkse updates en validatie door juridische experts.
- **Scalability**: Cloud hosting en vector databases voor groei.
- **Legal Compliance**: Disclaimers en bronvermelding voor juridische aansprakelijkheid.
```

    file.write(prd_content_part1 + prd_content_part2)

file_path
