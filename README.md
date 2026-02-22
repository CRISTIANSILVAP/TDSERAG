
# TDSERAG — Implementación Completa de RAG con LangChain

Este proyecto implementa paso a paso un sistema **RAG (Retrieval-Augmented Generation)** utilizando LangChain y múltiples proveedores de modelos (Ollama, OpenAI, Anthropic y Gemini).

El notebook muestra la evolución desde un pipeline básico de indexación hasta un **RAG Agent con memoria conversacional**, permitiendo entender tanto la arquitectura clásica como una arquitectura más avanzada basada en agentes.

---

# ¿Qué es RAG?

**Retrieval-Augmented Generation (RAG)** es un patrón que combina:

1. **Recuperación de información (Retrieval)** desde una base de conocimiento.
2. **Generación de texto (LLM)** utilizando el contexto recuperado.

Flujo general:

```
Pregunta → Retriever → Contexto relevante → LLM → Respuesta fundamentada
```

El modelo no responde únicamente con su conocimiento interno, sino apoyándose en información externa previamente indexada.

---

# Estructura del Notebook

El notebook está dividido en las siguientes secciones:

---

## 0️ Instalación de dependencias

Se instalan las librerías necesarias para:

* Construcción de RAG con LangChain
* Uso de múltiples proveedores de LLM
* Procesamiento de texto
* Manejo de memoria con LangGraph

Incluye soporte para:

* langchain
* langchain-community
* langgraph
* langchain-openai
* langchain-ollama
* langchain-anthropic
* langchain-google-genai
* python-dotenv
* beautifulsoup4

---

## 1️ Configuración del entorno

Se cargan variables desde `.env` o `apikey.env`.

Variables principales:

| Variable          | Descripción                                              |
| ----------------- | -------------------------------------------------------- |
| `LLM_PROVIDER`    | Proveedor del modelo (ollama, openai, anthropic, gemini) |
| `LLM_MODEL`       | Nombre del modelo a usar                                 |
| `LLM_TEMPERATURE` | Temperatura del modelo                                   |
| `SOURCE_URL`      | URL que se va a indexar                                  |
| `CHUNK_SIZE`      | Tamaño de fragmentos                                     |
| `CHUNK_OVERLAP`   | Superposición entre fragmentos                           |
| `TOP_K`           | Número de documentos recuperados                         |

Incluye validaciones y protección de claves sensibles.

---

## 2️ Inicialización del modelo y embeddings

Según el proveedor configurado:

* Se instancia el modelo de chat.
* Se instancia el modelo de embeddings.
* Se valida que el modelo de embeddings funcione correctamente.

Soporta ejecución local con Ollama o proveedores en la nube.

---

## 3️ Indexación — Carga de documentos

Se utiliza un loader web para:

* Extraer contenido desde una URL.
* Filtrar únicamente contenido relevante (títulos, headers, contenido principal).
* Preparar el texto para procesamiento posterior.

---

## 4 Indexación — Chunking

El texto se divide usando un splitter recursivo que:

* Mantiene coherencia semántica.
* Controla tamaño de fragmento.
* Aplica overlap para evitar pérdida de contexto.

Esto optimiza la calidad del retrieval.

---

## Vector Store en memoria

Se genera el flujo:

```
Chunks → Embeddings → Vector Store
```

Los embeddings se almacenan en memoria para realizar búsquedas semánticas rápidas.

---

# RAG Chain (Implementación Clásica)

Se construye un pipeline RAG tradicional:

1. El usuario hace una pregunta.
2. El retriever obtiene los documentos más relevantes.
3. Se construye un prompt con el contexto.
4. Se realiza una única llamada al LLM.
5. Se devuelve la respuesta final.

Ventajas:

* Flujo simple
* Bajo costo
* Determinístico
* Fácil de debuggear

---

# RAG Agent con Tool y Memoria

Se implementa una versión más avanzada basada en agentes.

## 🔧 Tool personalizada

Se define una tool de recuperación que el agente puede invocar cuando lo considere necesario.

A diferencia del RAG chain, el agente:

* Decide si necesita contexto.
* Puede llamar la herramienta múltiples veces.
* Realiza razonamiento intermedio.

---

##  Memoria Conversacional

Se incorpora memoria usando un sistema de almacenamiento en memoria con identificador de sesión (`thread_id`).

Esto permite:

* Mantener contexto entre mensajes.
* Simular conversaciones persistentes.
* Manejar múltiples sesiones independientes.

---

#  RAG Chain vs RAG Agent

| Característica        | RAG Chain | RAG Agent |
| --------------------- | --------- | --------- |
| Llamadas al LLM       | 1         | Variable  |
| Decisión de retrieval | Fija      | Dinámica  |
| Memoria               | No        | Sí        |
| Complejidad           | Baja      | Media     |
| Costo                 | Bajo      | Mayor     |

---

#  Flujo Completo del Sistema

```
1. Cargar configuración
2. Inicializar modelo y embeddings
3. Cargar documentos desde web
4. Dividir en chunks
5. Crear embeddings
6. Guardar en vector store
7. Crear retriever
8. Construir RAG chain
9. Construir RAG agent
10. Ejecutar consultas
```

---

#  Objetivo del Proyecto

Este notebook está diseñado para:

* Entender RAG desde cero.
* Implementar retrieval real.
* Comparar arquitecturas (chain vs agent).
* Aprender manejo de memoria conversacional.
* Construir una base sólida para sistemas productivos.

---

#  Cómo Ejecutarlo

1. Crear archivo `.env` con las variables necesarias.
2. Instalar dependencias.
3. Ejecutar el notebook en orden.
4. Probar consultas tanto en:

   * RAG Chain
   * RAG Agent

---

# Posibles Extensiones

* Persistencia con FAISS o Chroma
* Ingesta de múltiples documentos
* RAG híbrido (BM25 + embeddings)
* Evaluación automática (RAGAS)
* Streaming de respuestas
* Agentes con múltiples tools

---

# Conceptos Cubiertos

* Embeddings
* Vector stores
* Chunking estratégico
* Retrieval top-k
* Prompt engineering para RAG
* Tool calling
* Arquitectura de agentes
* Memoria conversacional
* Soporte multi-proveedor de LLM

---

#  Conclusión

Este proyecto implementa un sistema RAG completo que evoluciona desde:

```
Indexación básica
     ↓
RAG Chain
     ↓
RAG Agent con memoria
```

Sirve como base para construir:

* Chatbots con conocimiento externo
* Asistentes técnicos
* Sistemas de preguntas y respuestas empresariales
* Agentes inteligentes con herramientas

