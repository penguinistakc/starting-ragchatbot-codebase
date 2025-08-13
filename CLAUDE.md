# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Setup and Dependencies
```bash
uv sync                           # Install Python dependencies
```

### Running the Application
```bash
./run.sh                         # Quick start (recommended)
cd backend && uv run uvicorn app:app --reload --port 8000  # Manual start
```

### Environment Setup
- Create `.env` file in root with `ANTHROPIC_API_KEY=your_key_here`
- Application serves at http://localhost:8000
- API docs available at http://localhost:8000/docs

## Architecture Overview

This is a **RAG (Retrieval-Augmented Generation) System** for course materials with a FastAPI backend and vanilla JavaScript frontend.

### Core Flow
1. **Document Processing**: Course documents from `docs/` → chunked → ChromaDB vectors
2. **Query Processing**: User query → AI tool calling → vector search → Claude response
3. **Session Management**: Conversation context maintained across queries

### Key Architecture Patterns

**Tool-Based Search**: The system uses Anthropic's tool calling where Claude decides when to search:
- `search_tools.py`: Defines `CourseSearchTool` with semantic course name matching
- `ai_generator.py`: Handles tool execution flow with Claude API
- Search results include source attribution for UI display

**Dual Vector Collections**:
- `course_catalog`: Course metadata for semantic course name resolution
- `course_content`: Actual chunked course material for content search

**Component Organization**:
- `rag_system.py`: Main orchestrator coordinating all components
- `vector_store.py`: ChromaDB interface with unified search method
- `document_processor.py`: Text chunking and course parsing
- `session_manager.py`: Conversation history management
- `models.py`: Pydantic models for Course, Lesson, CourseChunk

### Data Models
- **Course**: Title, instructor, lessons list, course link
- **Lesson**: Number, title, optional lesson link
- **CourseChunk**: Content, course title, lesson number, chunk index

### Configuration
- `config.py`: Centralized settings with environment variable loading
- Key settings: chunk size (800), overlap (100), max results (5), max history (2)
- Uses `claude-sonnet-4-20250514` model and `all-MiniLM-L6-v2` embeddings

### Frontend-Backend Communication
- Frontend (`script.js`) sends POST to `/api/query` with `{query, session_id}`
- Backend returns `{answer, sources, session_id}` 
- Sources displayed in collapsible UI sections
- Session persistence maintains conversation context

### Document Loading
- Startup: Automatically loads `.pdf`, `.docx`, `.txt` files from `docs/`
- Avoids duplicate processing by checking existing course titles
- Course analytics available via `/api/courses` endpoint
- always use uv to run server do not use pip directly
- make sure to use uv to manage all dependencies
- use uv to run python files