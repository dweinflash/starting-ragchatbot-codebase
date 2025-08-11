# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Development Commands

```bash
# Start the application (development mode)
./run.sh

# Alternative manual startup
cd backend && uv run uvicorn app:app --reload --port 8000

# Install/sync dependencies
uv sync

# Set up environment (required)
echo "ANTHROPIC_API_KEY=your_key_here" > .env
```

## Application URLs
- Main interface: http://localhost:8000
- API docs: http://localhost:8000/docs  
- Analytics dashboard: http://localhost:8000/visualization.html

## Core Architecture

This is a **Retrieval-Augmented Generation (RAG) system** that answers questions about course materials using semantic search and AI generation.

### RAG Pipeline Flow
1. **Document Processing** → Course docs chunked and embedded into ChromaDB
2. **Query Processing** → User questions trigger tool-based semantic search
3. **Context Retrieval** → Relevant chunks retrieved via vector similarity
4. **AI Generation** → Claude generates responses using retrieved context
5. **Session Management** → Conversation history maintained per session

### Key Components

**Backend (`backend/`)**
- `rag_system.py` - Main orchestrator coordinating all components
- `vector_store.py` - ChromaDB integration with dual collections (catalog + content)
- `document_processor.py` - Intelligent chunking with course structure parsing
- `ai_generator.py` - Claude integration with tool execution framework
- `search_tools.py` - Pluggable tool architecture for search operations
- `session_manager.py` - Conversation state management
- `app.py` - FastAPI REST API with static file serving

**Frontend (`frontend/`)**
- Vanilla JavaScript with modern async patterns
- Split-pane UI: sidebar (course stats, suggestions) + chat area
- Markdown rendering via Marked.js
- Analytics dashboard using D3.js for visualizations

### Data Storage
- **ChromaDB** (persistent, `./chroma_db/`)
  - `course_catalog` collection - course metadata and titles
  - `course_content` collection - chunked content with embeddings
- **Sessions** - In-memory conversation history (lost on restart)
- **Documents** - Course files in `/docs/` auto-loaded on startup

### Configuration (`backend/config.py`)
- **Required**: `ANTHROPIC_API_KEY` environment variable
- **AI Model**: claude-sonnet-4-20250514
- **Embeddings**: all-MiniLM-L6-v2 (sentence-transformers)
- **Chunking**: 800 chars with 100 char overlap
- **Search**: Top 5 results, 2-exchange conversation memory

## Document Format Requirements

Course documents must follow this structure:
```
Course Title: [Title]
Course Link: [URL]  
Course Instructor: [Name]

Lesson 0: [Title]
Lesson Link: [URL]
[Content...]

Lesson 1: [Title]
[Content...]
```

## API Endpoints

- `POST /api/query` - Process user questions with RAG pipeline
- `GET /api/courses` - Return course statistics and titles
- Static files served from `/frontend/`

## Advanced Features

### Semantic Search
- Course name resolution (partial matches work)
- Multi-level filtering (course + lesson specific)
- Context-aware chunking preserves lesson boundaries
- Source attribution links results to original content

### Tool Architecture
- AI dynamically invokes search tools based on user queries
- Abstract tool classes allow easy extension
- Tools can be chained and composed
- Search results include source tracking

### Session Management
- Multi-turn conversations with context retention
- Configurable history limits (default: 2 exchanges)
- Session-scoped memory prevents cross-contamination

## Development Notes

- **Hot reload** enabled for backend development
- **No-cache headers** for frontend assets during development
- **CORS enabled** for cross-origin development
- Uses **uv** for Python package management
- Requires **Python 3.13+**
- **FastAPI** provides automatic API documentation at `/docs`

## Extension Points

- Add new tools by extending `SearchTool` base class
- Support additional document formats in `document_processor.py`
- Replace ChromaDB with other vector stores via `VectorStore` interface
- Add persistent session storage (currently in-memory)
- Implement user authentication and multi-tenancy
- always use uv to run the server do not use pip directly
- make sure to use uv to manage all dependencies
- use uv to run python files