# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Start Commands

### Development Server
```bash
# Start the RAG system server
./run.sh
# Or manually:
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Environment Setup
```bash
# Install dependencies
uv sync

# Set up environment variables
cp .env.example .env
# Edit .env to add your ANTHROPIC_API_KEY
```

### Access Points
- Web Interface: http://localhost:8000
- API Documentation: http://localhost:8000/docs

## Architecture Overview

This is a **Retrieval-Augmented Generation (RAG) system** for querying course materials using semantic search and AI-powered responses.

### Core Components Flow
```
Query → RAGSystem → VectorStore (ChromaDB) → AIGenerator (Claude) → Response
```

### Key Architecture Patterns

**Central Orchestrator**: `RAGSystem` (`backend/rag_system.py`) coordinates all components and manages the query processing pipeline.

**Dual Vector Storage**: Two ChromaDB collections managed by `VectorStore`:
- `course_catalog`: Course metadata, instructor info, lesson structure
- `course_content`: Text chunks with semantic embeddings

**Document Processing Pipeline**: 
1. `DocumentProcessor` parses structured course files with headers: `Course Title:`, `Course Link:`, `Course Instructor:`
2. Extracts lessons marked as `Lesson N: Title`
3. Creates overlapping text chunks with configurable size/overlap
4. Enriches chunks with context: `"Course [title] Lesson [N] content: [chunk]"`

**Tool-Based AI**: `AIGenerator` uses Anthropic's tool calling with `CourseSearchTool` for semantic search integration.

**Session Management**: `SessionManager` maintains conversation history with configurable message limits.

## Configuration System

All settings centralized in `backend/config.py`:
- **Chunking**: `CHUNK_SIZE=800`, `CHUNK_OVERLAP=100`
- **Search**: `MAX_RESULTS=5` 
- **AI Model**: `claude-sonnet-4-20250514`
- **Embeddings**: `all-MiniLM-L6-v2` (sentence-transformers)
- **Storage**: `./chroma_db` for persistent vector store

## Document Format

Expected course document structure:
```
Course Title: [title]
Course Link: [url] 
Course Instructor: [instructor]

Lesson 0: Introduction
Lesson Link: [optional lesson url]
[lesson content]

Lesson 1: Next Topic
[lesson content]
```

## Data Flow

1. **Ingestion**: Course documents → `DocumentProcessor` → structured chunks
2. **Storage**: Chunks → `VectorStore` → ChromaDB collections
3. **Query**: User question → semantic search → relevant chunks
4. **Generation**: Context + query → `AIGenerator` → Claude API → response
5. **Session**: Conversation history maintained across queries

## Key Dependencies

- **uv**: Python package management and runtime
- **FastAPI**: Web framework with auto-generated docs
- **ChromaDB**: Vector database for semantic search
- **sentence-transformers**: Text embeddings
- **anthropic**: Claude API client
- **python-dotenv**: Environment variable management

## Development Notes

- No test framework currently configured
- No linting/formatting tools configured  
- ChromaDB data persisted in `backend/chroma_db/` (gitignored)
- Frontend is basic HTML/CSS/JS in `frontend/` directory
- Course materials stored in `docs/` as `.txt` files
- Environment variables loaded from `.env` (use `.env.example` as template)
- make sure to use uv to manage all dependencies