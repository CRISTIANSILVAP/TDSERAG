
# TDSERAG — RAG (sobre 1 artículo) con LangChain

Este repo es un **notebook educativo** que arma un flujo RAG completo (indexing + retrieval + generación) y lo aplica a **un solo texto fuente**.

Por defecto el texto que se indexa es el post de Lilian Weng:
https://lilianweng.github.io/posts/2023-06-23-agent/

La implementación principal está en **RAG.ipynb**.

---

## Qué hace exactamente

El notebook sigue el pipeline clásico:

1) **Indexing**
	- Cargar HTML desde una URL
	- Limpiar/filtrar contenido relevante (título/headers/cuerpo)
	- Partir en chunks (con overlap)
	- Embeddings
	- Guardar en un vector store en memoria

2) **Retrieval + Generation (RAG chain)**
	- Busca los top-k chunks relevantes
	- Hace **una llamada** al LLM con (pregunta + contexto)

3) **RAG agent (con tool de retrieval)**
	- Expone una tool `retrieve_context` para que el agente decida cuándo recuperar contexto
	- Incluye un **fallback** si:
	  - no corriste la celda que inicializa el agente, o
	  - tu modelo (por ejemplo en Ollama) no soporta tool calling

Importante: en todos los modos, la instrucción es **responder usando el contexto recuperado**. Si la respuesta no está en el artículo, lo correcto es que responda tipo *“no sé / no está en el contexto”*.

---

## Estructura

- `RAG.ipynb`: notebook principal
- `requirements.txt`: dependencias Python
- `apikey.env`: configuración local (provider/model/URL)
- `README.md`: esta guía

---

## Requisitos

### 1) Python

- Windows
- Python 3.10+ (recomendado 3.11)

### 2) Ollama (para correr todo local)

Si usas `LLM_PROVIDER=ollama` (por defecto), necesitas **dos cosas**:

1) La app/servicio de Ollama instalado (esto es lo que levanta `http://localhost:11434`).
2) Los modelos descargados (chat + embeddings).

Ojo: `pip install ollama` **no reemplaza** instalar Ollama Desktop. El paquete de Python es para cliente, pero el server tiene que estar corriendo.

Descarga: https://ollama.com/download

---

## Setup rápido (Windows + PowerShell)

### 1) Crear y activar venv

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

### 2) Instalar dependencias

```powershell
python -m pip install -r requirements.txt
```

### 3) Instalar/arrancar Ollama

1) Instala la app de Ollama.
2) Asegúrate de que el servicio está arriba.

Comandos útiles:

```powershell
ollama --version
ollama serve
```

Tip: `ollama serve` se queda “corriendo” (no termina). Si lo ejecutas así, déjalo en una terminal y usa otra para seguir con VS Code/Jupyter.

Si `ollama` no aparece en tu PATH, puedes llamarlo por ruta (común en Windows):

```powershell
$ollama = "$env:LOCALAPPDATA\Programs\Ollama\ollama.exe"
& $ollama --version
& $ollama serve
```

### 4) Descargar modelos (chat + embeddings)

En PowerShell:

```powershell
ollama pull llama3
ollama pull nomic-embed-text
ollama list
```

Notas:
- El notebook usa tags tipo `llama3:latest` (lo que te salga en `ollama list`).
- Si estás en Git Bash/WSL y te da `command not found`, corre estos comandos desde **PowerShell**.

---

## Configuración (apikey.env)

El notebook carga variables desde `.env` si existe, y además carga `apikey.env` si existe (en este repo lo estamos usando como archivo principal de config).

Si vas a poner API keys reales acá: **no las subas al repo**. Lo normal es:

- Tener `apikey.env` local y agregarlo al `.gitignore`.
- O usar variables de entorno del sistema.

Variables más importantes:

- `LLM_PROVIDER`: `ollama` | `anthropic` | `openai` | `gemini`
- `LLM_MODEL`:
  - Ollama: el nombre exacto de `ollama list` (ej: `llama3:latest`)
- `OLLAMA_EMBED_MODEL`: ej `nomic-embed-text`
- `SOURCE_URL`: URL a indexar (por defecto el post de Lilian Weng)
- `TOP_K`: cuántos chunks recuperar (ej 4)
- `CHUNK_SIZE` / `CHUNK_OVERLAP`: chunking

Ejemplo (modo 100% local):

```env
LLM_PROVIDER=ollama
LLM_MODEL=llama3:latest
OLLAMA_EMBED_MODEL=nomic-embed-text
SOURCE_URL=https://lilianweng.github.io/posts/2023-06-23-agent/
TOP_K=4
CHUNK_SIZE=1000
CHUNK_OVERLAP=200
```

Si cambias a Anthropic/OpenAI/Gemini, además necesitas la API key correspondiente:

- Anthropic: `ANTHROPIC_API_KEY=...`
- OpenAI: `OPENAI_API_KEY=...`
- Gemini: `GOOGLE_API_KEY=...`

---

## Cómo correr el notebook (orden recomendado)

Abre `RAG.ipynb` en VS Code (Jupyter) y ejecuta las celdas en este orden:

1) **Instalación** (solo la primera vez por kernel)
2) **Config** (carga `apikey.env`)
3) **Init** (modelo de chat + embeddings)
4) **Preflight embeddings** (valida que Ollama responde)
5) **Indexing**: loader → split → vector store
6) **RAG chain** (pregunta y responde con contexto)
7) **RAG agent** (tool + memoria) + celda de ejecución

Tip: si corres la última celda sin haber creado `vector_store`/`model`, el notebook te va a decir qué sección te falta correr.

---

## “Que lo que pregunte sea afín al texto”

La idea aquí es simple:

- El sistema siempre intenta responder **solo usando contexto recuperado del artículo**.
- Si preguntas algo que **no está** en el post, lo esperado es que te responda *“no sé / no está en el contexto recuperado”*.

Si quieres resultados más “estrictos” (rechazar preguntas fuera del tema en vez de intentar responder), el punto donde se ajusta es el prompt del sistema y/o una validación previa del retrieval (antes de llamar al LLM).

---

## Troubleshooting (lo típico)

### 1) “No pude usar Ollama para embeddings”

Checklist:

- Ollama instalado y corriendo (`ollama serve`)
- Modelo de embeddings descargado:

```powershell
ollama pull nomic-embed-text
ollama list
```

### 2) `ollama pull ...` falla en Git Bash/WSL

Usa PowerShell. En Windows, a veces `ollama` solo está disponible como `.exe` en `%LOCALAPPDATA%`.

### 3) El agent revienta con “does not support tools”

Eso pasa con algunos modelos de Ollama: el modo agent usa tool calling.

Soluciones:

- Dejar que el notebook use el **fallback** (retrieval explícito + 1 llamada al LLM), o
- Cambiar a un modelo que sí soporte tools, o
- Usar Anthropic/OpenAI para el chat model.

### 4) Respuestas “I don't know” aunque la info sí está

Prueba:

- Subir `TOP_K` (por ejemplo 6 u 8)
- Ajustar `CHUNK_SIZE`/`CHUNK_OVERLAP`
- Ver qué está devolviendo el retriever (hay una celda que imprime chunks)

---

## Notas

- Este repo es para aprender el flujo. El vector store es en memoria (no persiste).
- Si activas LangSmith (`LANGSMITH_TRACING=true`) puedes ver trazas, pero no es necesario.

