# 🏎️ Formula 101 — Agente Inteligente de Fórmula 1

Proyecto final de la materia **Sistemas Inteligentes** (UNAL).  
Agente conversacional especializado en F1 construido con **LangGraph**, **LangChain** y **RAG**.

---

## Arquitectura

```
Usuario
  │
  ▼
Streamlit (app.py)  ←── streaming token a token con cursor ▌
  │
  ▼
AgentStream → LangGraph ReAct Agent (Gemini 3.1 Flash Lite)
  │
  ├── Tool: get_latest_session ────────────► OpenF1 API (tiempo real)
  ├── Tool: get_race_positions              https://api.openf1.org/v1
  ├── Tool: get_pit_stops
  ├── Tool: get_lap_times
  ├── Tool: get_weather
  ├── Tool: get_drivers_in_session
  │
  ├── Tool: get_driver_standings ──────────► Jolpica API (histórico)
  ├── Tool: get_constructor_standings        https://api.jolpi.ca/ergast/f1
  ├── Tool: get_race_results
  ├── Tool: get_season_schedule
  ├── Tool: get_driver_career
  ├── Tool: get_qualifying_results
  │
  └── Tool: search_f1_knowledge ──────────► ChromaDB (RAG local)
                                             data/f1_knowledge/*.txt
```

| Componente | Tecnología | Rol |
|---|---|---|
| LLM | Gemini 3.1 Flash Lite | Razonamiento y generación |
| Agente | LangGraph `create_react_agent` | Ciclo ReAct: razona → tool → observa → responde |
| Embeddings | Google `text-embedding-004` | Vectorización de documentos F1 |
| Vector Store | ChromaDB | Búsqueda semántica en base de conocimiento |
| UI | Streamlit | Chat con streaming en tiempo real |
| APIs externas | OpenF1 + Jolpica | Datos en vivo e históricos (ambas gratuitas) |

---

## Setup

### 1. Instalar dependencias

```bash
pip install -r requirements.txt
```

### 2. Configurar API Key

```bash
cp .env.example .env
# Edita .env y agrega: GOOGLE_API_KEY=AIza...
```

Obtén tu API key gratis en: https://aistudio.google.com/apikey

### 3. Lanzar

```bash
streamlit run app.py
```

La primera vez construye automáticamente la base RAG (~15 seg). Las siguientes veces carga en caché.

---

## Optimizaciones de rendimiento

| Optimización | Implementación |
|---|---|
| **Streaming** | `AgentStream` itera chunks de LangGraph en tiempo real; la UI muestra tokens con cursor `▌` mientras el agente piensa |
| **Cache TTL** | `_TTLCache` en `tools.py` — 60 s para datos en vivo, 1 h para históricos. Segunda pregunta sobre la misma carrera es instantánea |
| **Embeddings ligeros** | `text-embedding-004` (Google API) — sin `torch` ni descarga de modelos locales (~500 MB menos) |
| **Descripciones cortas** | Docstrings de tools reducidos a 1 línea — ~30 % menos tokens de entrada por llamada LLM |

---

## Estructura del proyecto

```
Formula101/
├── app.py                        # Interfaz Streamlit
├── requirements.txt
├── .env.example
├── agent/
│   ├── graph.py                  # LangGraph ReAct agent + clase AgentStream
│   ├── tools.py                  # 13 tools con cache TTL
│   ├── rag.py                    # Pipeline RAG (Google embeddings + ChromaDB)
│   └── prompts.py                # System prompt del agente
└── data/
    ├── chroma_db/                # Vector store persistente (auto-generado)
    └── f1_knowledge/
        ├── equipos_pilotos.txt
        ├── historia_f1.txt
        ├── reglamentos_tecnicos.txt
        └── estrategia_analisis.txt
```

> Para agregar documentos propios: coloca archivos `.txt` en `data/f1_knowledge/` y elimina `data/chroma_db/` — se regenera automáticamente en el siguiente inicio.

---

## Tecnologías

- [LangGraph](https://langchain-ai.github.io/langgraph/) — framework de agentes
- [LangChain](https://python.langchain.com/) — integración LLM y tools
- [Google Gemini](https://ai.google.dev/) — modelo de lenguaje y embeddings
- [ChromaDB](https://www.trychroma.com/) — vector store local
- [OpenF1 API](https://openf1.org/) — datos F1 en tiempo real (gratuita, sin auth)
- [Jolpica API](https://jolpi.ca/) — datos históricos F1 desde 1950 (gratuita, sin auth)
- [Streamlit](https://streamlit.io/) — interfaz web
