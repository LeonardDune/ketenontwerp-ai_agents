# Product Requirements Document (PRD): AI Legal Advisor Agent
**Project Naam**: AI Legal Advisor Agent
**Versie**: 1.0
**Datum**: 17 september 2025
**Auteurs**: Renzo Wouters, BMAD Analyst Team

---

## 1. Goals and Background Context

### 1.1 Doel
Bouw een **AI Legal Advisor Agent** die juridisch advies kan geven op basis van **Nederlandse wetgeving en jurisprudentie**, met een **MVP-focus op strafrecht**. Het systeem moet later uitbreidbaar zijn naar andere rechtsgebieden (bijv. bestuurlijk recht, civiel recht).

### 1.2 Probleemstelling
- Juridische informatie is versnipperd over **wetten.nl** (wetgeving) en **rechtspraak.nl** (jurisprudentie).
- Het is moeilijk voor niet-juristen (bijv. burgers, kleine organisaties) om te bepalen of hun handelen **binnen de wet** valt.
- Handmatig zoeken is tijdrovend en foutgevoelig, vooral als **wetgeving en jurisprudentie** beide moeten worden geanalyseerd.

### 1.3 Background Context
- **Wetten.nl** biedt **XML-bestanden** van alle Nederlandse wetten (bijv. Wetboek van Strafrecht).
- **Rechtspraak.nl** biedt een **Open Data API** voor uitspraken (bijv. vonnissen in strafzaken).
- **Vector databases** (FAISS, Weaviate) en **embeddings** (OpenAI, Nomic) maken **semantische zoekopdrachten** mogelijk, wat essentieel is voor het vinden van relevante juridische teksten.

---

## 2. Requirements

### 2.1 Functional Requirements (FR)
| ID  | Beschrijving                                                                                     | Prioriteit |
|-----|---------------------------------------------------------------------------------------------------|------------|
| FR1 | De agent moet **wetten en artikelen** kunnen opzoeken op basis van een natuurlijke-taalvraag.      | Hoog       |
| FR2 | De agent moet **relevante jurisprudentie** (uitspraken) kunnen vinden en koppelen aan wetten.       | Hoog       |
| FR3 | De agent moet een **juridisch onderbouwd antwoord** geven met verwijzingen naar bronnen.          | Hoog       |
| FR4 | De agent moet **uitleggen waarom** een bepaald artikel of uitspraak relevant is.                   | Hoog       |
| FR5 | De agent moet **actuele wetgeving en jurisprudentie** gebruiken (automatische updates).            | Middel     |
| FR6 | Het systeem moet **uitbreidbaar** zijn naar andere rechtsgebieden (bijv. bestuurlijk recht).      | Laag       |

### 2.2 Non-Functional Requirements (NFR)
| ID   | Beschrijving                                                                                     | Metriek                          |
|------|---------------------------------------------------------------------------------------------------|----------------------------------|
| NFR1 | Antwoorden moeten binnen **5 seconden** gegenereerd worden.                                      | Responstijd < 5s                |
| NFR2 | De nauwkeurigheid van de antwoorden moet **minimaal 90%** zijn (gemeten via testcases).          | 90% correcte antwoorden         |
| NFR3 | Het systeem moet **schaalbaar** zijn voor 100+ gelijktijdige gebruikers.                         | 100+ requests/sec                |
| NFR4 | Alle antwoorden moeten **traceerbaar** zijn naar bronnen (URLs naar wetten.nl/rechtspraak.nl).   | 100% bronvermelding              |
| NFR5 | Het systeem moet **modulair** zijn, zodat nieuwe rechtsgebieden gemakkelijk kunnen worden toegevoegd. | < 1 dag werk per nieuw domein   |

---

## 3. User Interface Design Goals

### 3.1 Target Users
- **Primair**: Juridische professionals (bijv. advocaten, officieren van justitie) en **overheidsmedewerkers** (bijv. handhavers, beleidsmedewerkers).
- **Secundair**: Burgers die juridisch advies zoeken over **strafrechtelijke kwesties** (bijv. boetes, klachten).

### 3.2 Core Screens
1. **Startscherm**:
   - Veld om een juridische vraag in te voeren (bijv. *"Mag de politie mijn telefoon doorzoeken zonder toestemming?"*).
   - Dropdown om het **rechtsgebied** te selecteren (MVP: alleen "strafrecht").
2. **Resultatenscherm**:
   - Lijst met **relevante wetten en uitspraken**, gerangschikt op relevantie.
   - Filteropties (bijv. "Alleen wetten", "Alleen recentere uitspraken").
3. **Detailscherm**:
   - Volledige tekst van een wet/uitspraak + **uitleg van de agent** waarom dit relevant is.
   - **Bronvermelding** (URL naar wetten.nl/rechtspraak.nl).

### 3.3 Accessibility
- WCAG AA-compliant (bijv. voldoende contrast, toetsenbordnavigatie).
- **Eenvoudige taal** voor niet-juridische gebruikers (bijv. uitleg van juridische termen).

---

## 4. Technical Assumptions

### 4.1 Tech Stack
| Categorie          | Technologie               | Versie/Notities                                  |
|--------------------|---------------------------|--------------------------------------------------|
| **Backend**        | Python                    | 3.10+                                            |
| **Embeddings**     | OpenAI `text-embedding-3-large` | Of `nomic-embed-text` (open source).       |
| **Vector DB**      | FAISS                     | Of Weaviate/Pinecone voor schaalbaarheid.        |
| **APIs**           | wetten.nl (XML), rechtspraak.nl (Open Data) | Officiële bronnen.       |
| **Frontend**       | React/Next.js             | Voor de gebruikersinterface.                     |
| **Hosting**        | AWS/GCP                   | Voor schaalbaarheid en betrouwbaarheid.          |
| **CI/CD**          | GitHub Actions            | Voor automatische tests en deployments.         |

### 4.2 Repository Structure
project-root/
├── backend/
│   ├── scripts/               # Scripts voor het downloaden/parsen van wetten en uitspraken.
│   ├── retrieval/             # Logica voor semantische zoekopdrachten.
│   ├── ranking/               # Ranking-logica voor resultaten.
│   └── api/                   # API-endpoints voor de frontend.
├── frontend/
│   ├── components/            # React-componenten (bijv. zoekveld, resultatenlijst).
│   ├── pages/                 # Pagina's (bijv. startscherm, detailscherm).
│   └── styles/                # CSS/Styled Components.
├── data/
│   ├── wetten/                # Gedownloade XML-bestanden van wetten.nl.
│   ├── uitspraken/            # JSON-bestanden met uitspraken van rechtspraak.nl.
│   └── embeddings/            # FAISS-indexen met embeddings.
└── docs/                      # Documentatie (PRD, architectuur, etc.).


### 4.3 Coding Standards
- **Python**: PEP 8 + type hints.
- **Frontend**: TypeScript + ESLint.
- **Testing**:
  - Unit tests voor backend-functies (bijv. retrieval, ranking).
  - End-to-end tests voor de frontend (bijv. Cypress).
- **Documentatie**: Docstrings voor alle functies; README voor setup en gebruik.

---

## 5. Epic and Story Structure

### 5.1 Epic 1: Corpus en Retrieval Systeem (MVP: Strafrecht)
**Doel**: Bouw een systeem om **wetten en uitspraken** te downloaden, parsens, en indexeren, met focus op strafrecht.

| Story ID  | Beschrijving                                                                                     | Acceptatiecriteria                                                                                     |
|-----------|---------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| S1.1       | Als **ontwikkelaar** wil ik een script om **wetten van wetten.nl** (bijv. Wetboek van Strafrecht) te downloaden en te parsens, zodat we een up-to-date corpus hebben. | - Script downloadt XML-bestanden van wetten.nl. <br> - XML wordt geparst naar artikelen met metadata (bijv. artikelnummer, wet, URL). |
| S1.2       | Als **ontwikkelaar** wil ik een script om **uitspraken van rechtspraak.nl** (strafrecht) op te halen via de API, zodat we jurisprudentie kunnen integreren. | - Script haalt uitspraken op via de Open Data API. <br> - Uitspraken worden opgeslagen met metadata (bijv. ECLI, datum, rechtbank). |
| S1.3       | Als **ontwikkelaar** wil ik een **vector database** opzetten met embeddings van wetten en uitspraken, zodat we semantische zoekopdrachten kunnen doen. | - Embeddings gegenereerd met `text-embedding-3-large`. <br> - FAISS-index opgebouwd met metadata. |

### 5.2 Epic 2: Agent en Ranking Logica
**Doel**: Bouw de AI-agent die vragen beantwoordt met behulp van de geïndexeerde data, met focus op strafrecht.

| Story ID  | Beschrijving                                                                                     | Acceptatiecriteria                                                                                     |
|-----------|---------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| S2.1       | Als **gebruiker** wil ik een **natuurlijke-taalvraag** kunnen stellen (bijv. over strafrecht), zodat ik juridisch advies kan krijgen. | - Frontend heeft een invoerveld voor vragen. <br> - Vraag wordt doorgestuurd naar de backend. |
| S2.2       | Als **agent** wil ik **wetten en uitspraken** kunnen retrieven en rangschikken op relevantie, zodat ik nauwkeurige antwoorden kan geven. | - Retrieval-functie retourneert top-5 wetten/uitspraken. <br> - Ranking gebeurt op basis van relevantie, actualiteit, en citatie-boost. |
| S2.3       | Als **agent** wil ik **uitleggen waarom** een bepaald artikel of uitspraak relevant is, zodat de gebruiker het antwoord begrijpt. | - Antwoord bevat een samenvatting van de relevante passages. <br> - Antwoord bevat een uitleg van de agent (bijv. "Dit artikel is relevant omdat..."). |

### 5.3 Epic 3: Frontend en Gebruikersinterface
**Doel**: Bouw een gebruiksvriendelijke interface voor het stellen van vragen en het tonen van resultaten.

| Story ID  | Beschrijving                                                                                     | Acceptatiecriteria                                                                                     |
|-----------|---------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| S3.1       | Als **gebruiker** wil ik een **eenvoudig invoerveld** voor mijn vraag, zodat ik gemakkelijk advies kan vragen. | - Invoerveld is zichtbaar op het startscherm. <br> - Dropdown voor rechtsgebied (MVP: alleen "strafrecht"). |
| S3.2       | Als **gebruiker** wil ik **relevante wetten en uitspraken** zien, gerangschikt op relevantie, zodat ik de bronnen kan controleren. | - Resultatenlijst toont wetten en uitspraken met scores. <br> - Filteropties voor "Alleen wetten" of "Alleen uitspraken". |
| S3.3       | Als **gebruiker** wil ik een **duidelijke uitleg** van het antwoord, zodat ik weet waarom een bepaalde wet of uitspraak van toepassing is. | - Detailscherm toont volledige tekst + uitleg van de agent. <br> - Bronvermelding (URL) is aanwezig. |

### 5.4 Epic 4: Uitbreiding naar Andere Rechtsgebieden (Toekomst)
**Doel**: Maak het systeem uitbreidbaar naar andere rechtsgebieden (bijv. bestuurlijk recht, civiel recht).

| Story ID  | Beschrijving                                                                                     | Acceptatiecriteria                                                                                     |
|-----------|---------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| S4.1       | Als **ontwikkelaar** wil ik een **modulair ontwerp** voor het corpus, zodat nieuwe rechtsgebieden gemakkelijk kunnen worden toegevoegd. | - Corpus is opgedeeld per rechtsgebied (bijv. `data/wetten/strafrecht/`, `data/wetten/bestuursrecht/`). <br> - Scripts voor downloaden/parsen zijn herbruikbaar. |
| S4.2       | Als **ontwikkelaar** wil ik de **retrieval-logica** aanpassen om meerdere rechtsgebieden te ondersteunen. | - Retrieval-functie accepteert een `rechtsgebied`-parameter. <br> - Ranking-logica werkt voor alle rechtsgebieden. |

---

## 6. Checklist Results
Gebruik de `pm-checklist` om te valideren of alle requirements gedekt zijn. Voorbeelduitkomsten:
- **Functionele Eisen**: Alle FR's zijn gedekt door de epics/stories.
- **Technische Eisen**: De gekozen tech stack voldoet aan NFR1-NFR5.
- **Risico's**:
  - **Juridische aansprakelijkheid**: Gemitigeerd door disclaimers en bronvermelding.
  - **Data-actualiteit**: Opgelost door wekelijkse updates van het corpus.
  - **Schaalbaarheid**: FAISS/Weaviate en cloud-hosting zorgen voor schaalbaarheid.

---

## 7. Next Steps
1. **Architectuurdocument**: Maak een gedetailleerd architectuurdocument met:
   - Componentdiagrammen (bijv. hoe de backend communiceert met de vector databases).
   - API-specificaties (bijv. endpoints voor retrieval en ranking).
   - Dataflow-diagrammen (bijv. hoe een vraag wordt verwerkt).
2. **Prototype**: Bouw een proof-of-concept voor:
   - Het downloaden en indexeren van wetten/uitspraken (strafrecht).
   - De retrieval en ranking-logica.
3. **Testcases**: Definieer testcases om de nauwkeurigheid van de agent te valideren (bijv. 50 strafrechtelijke vragen met bekende antwoorden).
4. **Stakeholder Review**: Laat het PRD reviewen door:
   - Juridische experts (voor correctheid van de adviezen).
   - Ontwikkelaars (voor haalbaarheid van de technische implementatie).

---
