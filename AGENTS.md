# AGENTS.md - Odoo LLM Suite

**Generated**: 2026-06-06 | **Scale**: 753 files, ~53K LOC | **Modules**: 34

## OVERVIEW

LLM/AI integration suite for Odoo 18. Provider-agnostic framework for chat, embeddings, RAG, vector stores, function calling, and media generation. 34 modules spanning core framework, providers, knowledge, and business tools.

## STRUCTURE

```
odoo-llm/
├── llm/                    # Core: provider abstraction, dispatch, resource lifecycle
├── llm_thread/             # Chat threads, EventSource streaming, mail store
├── llm_tool/               # Function calling / tool execution framework
├── llm_assistant/          # Assistant config (model, params, authorized tools)
├── llm_store/              # Vector store abstraction
├── llm_knowledge/          # RAG: collections, chunking, embedding
├── llm_generate/            # Unified generation API (text + media)
├── llm_generate_job/        # Async job queue for heavy generation
├── llm_training/            # Fine-tuning dataset management
├── llm_mcp_server/          # Model Context Protocol server
├── web_json_editor/         # JSON editor widget for backend
├── [providers]/
│   ├── llm_openai/          # OpenAI (Chat, DALL-E, Embeddings, Training)
│   ├── llm_anthropic/       # Anthropic/Claude
│   ├── llm_mistral/         # Mistral AI
│   ├── llm_ollama/          # Ollama (local models)
│   ├── llm_letta/           # Letta/MemGPT
│   ├── llm_replicate/       # Replicate
│   ├── llm_fal_ai/          # Fal.ai (fast image generation)
│   ├── llm_comfyui/         # ComfyUI
│   └── llm_comfy_icu/       # ComfyICU cloud
├── [vector stores]/
│   ├── llm_pgvector/        # PostgreSQL pgvector
│   ├── llm_chroma/          # ChromaDB
│   └── llm_qdrant/          # Qdrant
├── [knowledge extensions]/
│   ├── llm_knowledge_automation/  # Auto-indexing RAG
│   ├── llm_knowledge_llama/       # LlamaIndex RAG
│   ├── llm_knowledge_mistral/     # Mistral RAG
│   └── llm_document_page/         # document.page RAG
└── [business tools]/
    ├── llm_tool_account/         # Accounting tools for LLM
    ├── llm_tool_demo/            # Demo CRM/Sale tools
    ├── llm_tool_knowledge/       # RAG tool for assistants
    ├── llm_tool_mis_builder/     # MIS Builder analysis
    ├── llm_tool_ocr_mistral/     # Mistral OCR
    ├── llm_tool_website/        # Website interaction
    └── account_invoice_import_llm/  # AI invoice extraction
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Provider dispatch | `llm/models/llm_provider.py` | `_dispatch('chat')` routes to provider module |
| Chat threads | `llm_thread/models/llm_thread.py` | 700 lines, EventSource, PostgreSQL locks |
| Tool execution | `llm_tool/models/llm_tool.py` | 569 lines, JSON schema, sandboxed execution |
| Assistant config | `llm_assistant/models/llm_assistant.py` | 546 lines |
| Knowledge collections | `llm_knowledge/models/llm_knowledge_collection.py` | 855 lines, chunking+embedding |
| Generation queue | `llm_generate_job/models/llm_generation_queue.py` | 528 lines, async jobs |
| OWL store service | `llm_thread/static/src/services/llm_store_service.js` | Mail store integration |
| MIS Builder tool | `llm_tool_mis_builder/models/mis_execution.py` | 713 lines |

## CONVENTIONS

- **Provider agnostic**: All business logic stays in `llm` core. Providers override `_dispatch` methods, never direct API calls.
- **Mail Store Odoo 18**: No `registerModel()`/`registerPatch()`. Use `Record` from `@mail/core/common/record` + `@web/core/utils/patch`.
- **Odoo 18 views**: `<list>` not `<tree>`, inline `invisible`/`readonly`/`required` not `attrs=`.
- **App Store HTML**: No RGBA colors, no CSS animations/transitions, no inline JS in `index.html`.
- **Dependencies**: Core chain = `llm` → `llm_thread` → `llm_tool` → `llm_assistant`.

## ANTI-PATTERNS

- **NEVER** instantiate provider API clients directly — use `_dispatch`
- **NEVER** use `registerModel()` or `registerPatch()` in JS — extend `Record` instead
- **NEVER** use `attrs=` or `<tree>` in XML views

## EXTERNAL DEPS

- `llm`: `requests`, `pydantic`
- `llm_openai`: `openai`
- `llm_anthropic`: `anthropic`
- `llm_pgvector`: `pgvector`
- `llm_chroma`: `chromadb`

## COMMANDS

```bash
# Update all LLM modules
python odoo_core/odoo/odoo-bin -c _conf/{db}.conf -d {db} -u llm,llm_thread,llm_tool,llm_assistant,llm_knowledge

# Test thread functionality
python odoo_core/odoo/odoo-bin -c _conf/{db}.conf -d {db} --test-enable -i llm_thread --stop-after-init
```

Find DB: `grep -l "singleflo/odoo-llm" _conf/*.conf` → prefer `*_dev18.conf`