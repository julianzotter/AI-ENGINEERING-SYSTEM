# AGENTS.md — AI-ENGINEERING-SYSTEM
# Repository: julianzotter/AI-ENGINEERING-SYSTEM
# Maintainer: Julian Zotter | ZOTTER LITSCHAUER ZT GMBH / ZOTTERCONSULT, Wien
# Stack: Python 3.11+, ChromaDB, Docker, BGE-M3 Embeddings, YAML-Orchestrierung
# Domäne: AEC (Architektur, Engineering, Construction) + AI/LLM Frameworks

## Projekt-Übersicht

Dieses Repository ist das zentrale AI-Engineering-Framework für:
- **Eurocode-Berechnungsmodule** (EC2 Beton, EC3 Stahl, EC5 Holz/HBV/TCC)
- **RAG/VectorDB-Pipeline** (ChromaDB on-premise, BGE-M3, 768D Embeddings)
- **Custom GPT Knowledge Bases** (KB-001 bis KB-999, 44 Ordner, ~299 Dateien)
- **YAML-gesteuerte Orchestrierung** mit Chain-of-Thought (CoT) Workflows
- **MAPE-K Adaptive Loop** für Self-Monitoring und Self-Healing
- **Multi-LLM Cognitive Collaboration** (VHW-IoC Architektur)

## Verzeichnisstruktur (Ziel-Architektur)

```
AI-ENGINEERING-SYSTEM/
├── AGENTS.md                        ← Codex-Anweisungen (diese Datei)
├── README.md                        ← Projektbeschreibung
├── OPTIMIZATION_GUIDE.md            ← Patterns: Retry, Circuit Breaker, Pydantic
├── requirements.txt                 ← Python-Abhängigkeiten
├── pyproject.toml                   ← Build-Konfiguration
├── docker/
│   ├── docker-compose.yml           ← Stack: ChromaDB + App + Monitoring
│   └── Dockerfile                   ← python:3.11-slim, multi-stage
├── src/
│   ├── eurocode/                    ← EC2/EC3/EC5 Berechnungsmodule
│   │   ├── hbv_calculator.py        ← Holz-Beton-Verbund (Gamma-Methode)
│   │   ├── tcc_analysis.py          ← Timber-Concrete Composite
│   │   ├── steel_ec3.py             ← Stahlbau nach EC3
│   │   └── concrete_ec2.py          ← Betonbau nach EC2
│   ├── rag/                         ← RAG-Pipeline + VectorDB
│   │   ├── chunker.py               ← 512-Token Chunks, 20% Overlap
│   │   ├── embedder.py              ← BGE-M3 (BAAI/bge-m3) 768D
│   │   ├── indexer.py               ← ChromaDB Index-Management
│   │   └── retriever.py             ← Hybrid-Retrieval (Dense + Sparse + RRF)
│   ├── orchestration/               ← YAML-Pipelines + CoT + MAPE-K
│   │   ├── pipeline_runner.py       ← YAML-Pipeline Executor
│   │   ├── cot_engine.py            ← Chain-of-Thought / Red Chain
│   │   └── mape_k.py               ← Monitor-Analyze-Plan-Execute-Knowledge
│   ├── gpt_registry/                ← Custom GPT Management
│   │   ├── registry.py              ← GPT-Registry (7 GPTs: 4 aktiv, 3 geplant)
│   │   └── health_check.py          ← KB-Verfügbarkeit, Embedding-Aktualität
│   └── utils/                       ← Shared Utilities
│       ├── config.py                ← YAML Config Loader
│       ├── logging_setup.py         ← structlog mit JSON-Output
│       └── validators.py            ← Input-Validierung, Einheiten-Check
├── knowledge_base/                  ← KB-Ordner (Symlinks oder Manifests)
├── pipelines/                       ← YAML-Pipeline-Definitionen
│   ├── rag_index.yaml               ← RAG-Indexierungs-Pipeline
│   └── eurocode_check.yaml          ← Berechnungs-Validierungs-Pipeline
├── materials/                       ← Materialkennwerte als YAML
│   ├── holz.yaml                    ← GL24h, GL28h, BSH, etc.
│   ├── beton.yaml                   ← C20/25, C25/30, etc.
│   └── stahl.yaml                   ← S235, S355, etc.
├── tests/                           ← pytest Test-Suite
│   ├── conftest.py                  ← Fixtures: Standardmaterialien
│   ├── test_hbv_calculator.py
│   ├── test_chunker.py
│   └── test_retriever.py
└── docs/                            ← Technische Dokumentation
```

## Befehle

```bash
# Setup
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# Tests
pytest -x --tb=short -q

# Linting + Typing
ruff check src/ tests/
mypy src/ --strict

# RAG-Pipeline
python -m src.rag.indexer --config pipelines/rag_index.yaml

# Docker-Stack
docker compose -f docker/docker-compose.yml up -d

# KB-Scanner
python -m src.utils.kb_scanner --root ./knowledge_base
```

## Code-Stil und Konventionen

### Python
- **Python 3.11+** als Minimum
- **Type-Hints überall** — mypy --strict kompatibel
- **Docstrings:** Google-Style, deutsch für AEC-Domain, englisch für generische Module
- **Variablen:** snake_case, sprechende Namen (nie `x`, `temp`, `data`)
- **Konstanten:** UPPER_SNAKE_CASE
- **Max. Funktionslänge:** 40 Zeilen, eine Verantwortung pro Funktion
- **Imports:** Immer am Dateianfang, keine * Imports, keine Any-Types

### Fehlerbehandlung
- Explizite Exceptions: `ValueError`, `TypeError`, domänenspezifisch
- Custom Exceptions: `EurocodeCalculationError`, `RAGIndexError`, `PipelineExecutionError`
- Fehlermeldungen: Klar, deutsch, mit Kontext — nie bare `except:`
- Keine stillen Fehler, keine Fallback-Defaults ohne explizite Anweisung

### Git
- Commit-Prefix: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`
- Branch-Schema: `feature/<kb-nr>-<beschreibung>` (z.B. `feature/kb-001-hbv-refactor`)
- Nie direkt auf `main` pushen — immer Branch + PR

### Testing
- **pytest** als Framework
- Mindestens 1 Test pro öffentliche Funktion
- Fixtures für Standardmaterialien in `conftest.py`
- Tests VOR Code-Änderungen ausführen

## Domain-Wissen (KRITISCH)

### Eurocode-Berechnungen
- **Einheitencheck ist Pflicht:** kN, kN/m², mm, cm — IMMER explizit in Variablennamen
  Beispiel: `moment_kNm`, `stress_MPa`, `length_mm`
- **Gamma-Methode (HBV):** `gamma_i ∈ [0, 1]`, `EI_eff` nach EC5 Annex B
- **Teilsicherheitsbeiwerte:** `gamma_M` aus YAML laden, NICHT hardcoden
- **Materialkennwerte:** Aus `materials/*.yaml` laden, nie direkt im Code
- **Jede Formel:** Kommentar mit Norm-Referenz: `# EC5, Gl. (B.1)` oder `# EC2 §6.1, Gl. (6.23)`
- **ÖNORM-Spezifika:** Nationale Anhänge (NA) als separate YAML-Overrides

### RAG/VectorDB
- **Chunks:** 512 Tokens, 20% Overlap, RecursiveCharacterTextSplitter
- **Embeddings:** BGE-M3 (BAAI/bge-m3), 768 Dimensionen, normiert
- **VectorDB:** ChromaDB persistent, Collection pro KB-Ordner
- **Schema:** `{id, embedding[768], text, metadata{source, kb_nr, chunk_idx, timestamp, doc_type}}`
- **Retrieval:** Hybrid (Dense Cosine + Sparse BM25), Reciprocal Rank Fusion
- **Similarity-Threshold:** 0.7 (Cosine), konfigurierbar über YAML

### Custom GPTs (Produktionsstatus)
| GPT | Status | KB |
|-----|--------|----|
| BRAND-GPT | ✅ Produktiv | KB-006 (2.4 MB JSON) |
| HBV/TCC-GPT | ✅ Produktiv | KB-001 |
| CAD-Parser-GPT (DZDXF) | ✅ Produktiv | KB-007 |
| LV-GPT | ✅ Produktiv | KB-020 |
| STAHL-GPT | ❌ Fehlend | KB-005 (Daten vorhanden) |
| HOLZ-GPT | ❌ Fehlend | KB-003 (Daten vorhanden) |
| BETON-GPT | ❌ Fehlend | KB-004 (Daten vorhanden) |

## Wichtige Regeln (DO / DON'T)

### DO
- ✅ Alle Normen-Parameter über YAML-Config laden
- ✅ Eurocode-Formeln mit Quellverweis kommentieren
- ✅ RAG-Chunks immer mit vollständiger Metadata versehen
- ✅ Structured Logging (structlog, JSON) für alle Module
- ✅ Docker-Images: `python:3.11-slim`, multi-stage builds
- ✅ Ergebnis-Dicts immer mit `unit`, `formula_ref`, `value` Feldern
- ✅ Vor jeder Änderung: bestehende Tests ausführen

### DON'T
- ❌ Keine hardcodierten Normparameter (gamma_M, f_ck, E_0_mean)
- ❌ Keine * Imports, keine Any-Types
- ❌ Keine stillen Fehler oder bare `except:`
- ❌ Kein direkter Push auf `main`
- ❌ Keine Cloud-Abhängigkeiten in der RAG-Pipeline (alles on-premise)
- ❌ Keine automatischen Index-Rebuilds ohne expliziten CLI-Befehl
- ❌ Keine Dateien >10 MB ins Repository committen
