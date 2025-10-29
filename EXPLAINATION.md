# LLM Graph Builder - Comprehensive Technical Documentation

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Backend Modules](#backend-modules)
4. [Frontend Components](#frontend-components)
5. [Data Flow](#data-flow)
6. [Key Features](#key-features)
7. [Deployment](#deployment)
8. [Configuration](#configuration)

---

## Project Overview

The **LLM Graph Builder** is a full-stack application designed to transform unstructured data into structured Knowledge Graphs stored in Neo4j. It leverages Large Language Models (LLMs) and the LangChain framework to extract entities, relationships, and properties from various data sources including PDFs, documents, YouTube videos, Wikipedia articles, and web pages.

### Technology Stack
- **Backend**: Python, FastAPI, LangChain, Neo4j
- **Frontend**: React, TypeScript, Vite
- **Database**: Neo4j Graph Database
- **LLM Integration**: OpenAI, Gemini, Anthropic, Groq, AWS Bedrock, Ollama, and more
- **Containerization**: Docker, Docker Compose

---

## Architecture

### High-Level Architecture

The application follows a client-server architecture with three main layers:

1. **Presentation Layer (Frontend)**: React-based UI for user interaction
2. **Application Layer (Backend)**: FastAPI server handling business logic
3. **Data Layer**: Neo4j graph database for storing knowledge graphs

```
┌─────────────────────────────────────────────────────────────┐
│                      Frontend (React)                        │
│  - File Upload UI    - Chat Interface                       │
│  - Graph Visualization - Settings Management                │
└──────────────────────┬──────────────────────────────────────┘
                       │ REST API / WebSocket
┌──────────────────────▼──────────────────────────────────────┐
│                   Backend (FastAPI)                          │
│  - Document Processing  - Entity Extraction                 │
│  - LLM Integration     - Graph Construction                 │
│  - QA System           - Communities Detection              │
└──────────────────────┬──────────────────────────────────────┘
                       │ Cypher Queries
┌──────────────────────▼──────────────────────────────────────┐
│                Neo4j Graph Database                          │
│  - Documents & Chunks  - Entities & Relationships           │
│  - Communities         - Embeddings                         │
└─────────────────────────────────────────────────────────────┘
```

---

## Backend Modules

The backend is built with Python and FastAPI, organized into specialized modules:

### 1. **score.py** - Main Application Entry Point
**Location**: `/backend/score.py`

**Functionality**:
- FastAPI application initialization and configuration
- CORS middleware setup for cross-origin requests
- Health check endpoints
- File upload handling with chunking support
- Security middleware (XContentTypeOptions, XFrameOptions)
- GZip compression for responses
- Session management
- Route definitions for all API endpoints

**Key Endpoints**:
- `/health` - Health check endpoint
- `/upload` - File upload with chunking
- `/merge` - Merge uploaded chunks
- `/extract` - Extract entities and relationships from documents
- `/chat_bot` - Chat interface for Q&A
- `/sources_list` - List available document sources
- `/delete_document_and_entities` - Delete documents and their entities

**Security Features**:
- Filename sanitization to prevent directory traversal attacks
- Input validation
- Secure file handling


### 2. **main.py** - Core Document Processing
**Location**: `/backend/src/main.py`

**Functionality**:
- Document source node creation for various sources (S3, GCS, local, YouTube, Wikipedia, web)
- Graph extraction from documents using LLMs
- Chunk processing and entity extraction
- Document status management
- Relationship creation between entities
- Post-processing and graph consolidation

**Key Functions**:

#### Document Source Handlers
- `create_source_node_graph_url_s3()`: Creates source nodes for S3 bucket files
- `create_source_node_graph_url_gcs()`: Creates source nodes for GCS bucket files
- `create_source_node_graph_web_url()`: Creates source nodes for web pages
- `create_source_node_graph_wikipedia_query()`: Creates source nodes for Wikipedia articles
- `create_source_node_graph_youtube_url()`: Creates source nodes for YouTube videos
- `create_source_node_graph_local_file()`: Creates source nodes for local files

#### Graph Extraction
- `extract_graph_from_file_local_file()`: Main extraction pipeline
  1. Retrieves document from Neo4j
  2. Loads document content based on source type
  3. Splits content into chunks
  4. Processes chunks to extract entities and relationships
  5. Creates graph structures in Neo4j
  6. Updates document status and statistics

**Processing Modes**:
- `START_FROM_BEGINNING`: Process document from scratch
- `START_FROM_LAST_PROCESSED_POSITION`: Resume from last chunk
- `DELETE_ENTITIES_AND_START_FROM_BEGINNING`: Clear existing entities and restart

### 3. **llm.py** - LLM Integration Module
**Location**: `/backend/src/llm.py`

**Functionality**:
- Centralized LLM model initialization
- Support for multiple LLM providers
- Graph transformation using LLM and Diffbot

**Supported LLM Providers**:

#### OpenAI Models
- GPT-4o, GPT-4o-mini, GPT-3.5, O3-mini
- Configuration: `LLM_MODEL_CONFIG_openai_<model>="model_name,api_key"`

#### Google Gemini
- Gemini 1.0 Pro, 1.5 Pro, 1.5 Flash
- Uses Google Cloud credentials
- Configurable safety settings to block harmful content

#### Azure OpenAI
- GPT-3.5, GPT-4o deployments
- Configuration: `LLM_MODEL_CONFIG_azure_<model>="model_name,endpoint,api_key,version"`

#### Anthropic Claude
- Claude 3.5 Sonnet
- Configuration: `LLM_MODEL_CONFIG_anthropic_<model>="model_name,api_key"`

#### Groq
- Llama 3 70B
- Configuration: `LLM_MODEL_CONFIG_groq_<model>="model_name,base_url,api_key"`

#### AWS Bedrock
- Claude models via AWS Bedrock
- Configuration: `LLM_MODEL_CONFIG_bedrock_<model>="model_name,access_key,secret_key,region"`

#### Ollama (Local LLMs)
- Llama 3 and other local models
- Configuration: `LLM_MODEL_CONFIG_ollama_<model>="model_name,local_url"`

#### Fireworks AI
- Various open-source models
- Configuration: `LLM_MODEL_CONFIG_fireworks_<model>="model_name,api_key"`

#### Diffbot
- Specialized graph extraction using Diffbot's NLP API
- Pre-trained for entity and relationship extraction

**Key Functions**:
- `get_llm(model)`: Returns configured LLM instance based on model name
- `get_graph_from_llm()`: Extracts graph structure from text using LLM
  - Uses `LLMGraphTransformer` for standard LLM models
  - Uses `DiffbotGraphTransformer` for Diffbot API


### 4. **graphDB_dataAccess.py** - Neo4j Database Operations
**Location**: `/backend/src/graphDB_dataAccess.py`

**Functionality**:
- Database access layer for Neo4j operations
- CRUD operations for documents, chunks, and entities
- Status tracking and error handling
- Transaction management

**Key Operations**:

#### Document Management
- `create_source_node()`: Creates Document nodes with metadata
  - File information (name, size, type, source)
  - Processing status
  - Model information
  - Chunk and entity counts
  - Cloud storage details (GCS, S3)

- `update_source_node()`: Updates document properties
  - Status updates (New, Processing, Completed, Failed, Cancelled)
  - Processing time tracking
  - Node and relationship counts
  - Error messages

- `get_current_status_document_node()`: Retrieves document status
- `delete_file_from_graph()`: Removes document and associated data

#### Chunk Operations
- `create_chunk_node()`: Creates Chunk nodes with text content
- `update_chunk_node()`: Updates chunk processing status
- `get_chunks()`: Retrieves chunks for a document
- `create_relationship_between_chunk_and_document()`: Links chunks to documents

#### Entity Management
- Entity nodes are created through graph extraction
- Tracks entity counts and relationships
- Supports entity embeddings for semantic search

#### Error Handling
- `update_exception_db()`: Logs errors to database
- Handles transient database errors with retry logic
- Cancellation status tracking

### 5. **QA_integration.py** - Question Answering System
**Location**: `/backend/src/QA_integration.py`

**Functionality**:
- Conversational QA interface over knowledge graphs
- Multiple retrieval strategies
- Context-aware responses with chat history
- Source attribution and metadata

**Chat Modes**:

#### 1. Vector Search Mode
- Uses vector embeddings for semantic similarity
- Retrieves relevant chunks based on query embedding
- Fast and efficient for large knowledge bases

#### 2. Graph Vector Mode
- Combines graph structure with vector search
- Retrieves entities and their relationships
- Provides richer context than pure vector search

#### 3. Graph Mode
- Uses Cypher queries generated by LLM
- Directly queries the graph structure
- Powered by `GraphCypherQAChain`

#### 4. Fulltext Search Mode
- Uses Neo4j fulltext indexes
- Keyword-based retrieval
- Good for exact term matching

#### 5. Hybrid Mode (Graph + Vector + Fulltext)
- Combines multiple retrieval strategies
- Reranks results for best matches
- Most comprehensive but computationally expensive

#### 6. Entity Vector Mode
- Searches entities using embeddings
- Retrieves entity-centric information

#### 7. Global Vector Mode
- Searches across community summaries
- Provides high-level overview responses

**Key Components**:

- `SessionChatHistory`: Manages conversation history per session
- `CustomCallback`: Tracks question transformation
- `get_total_tokens()`: Calculates token usage across different LLM providers
- Retrieval chains with contextual compression
- Document reranking for better results
- Source metadata extraction


### 6. **create_chunks.py** - Document Chunking
**Location**: `/backend/src/create_chunks.py`

**Functionality**:
- Splits documents into manageable chunks
- Maintains context with configurable overlap
- Handles different document types (PDFs, YouTube transcripts, etc.)

**Class: CreateChunksofDocument**

**Methods**:
- `split_file_into_chunks(token_chunk_size, chunk_overlap)`: Main chunking logic
  - Uses `TokenTextSplitter` from LangChain
  - Respects token limits
  - Preserves page numbers for PDFs
  - Handles YouTube timestamps
  - Creates LangChain Document objects

**Special Handling**:
- PDF documents: Includes page numbers in metadata
- YouTube videos: Includes timestamps and video segments
- Web pages and Wikipedia: Standard text chunking
- Configurable chunk size and overlap

### 7. **graph_query.py** - Graph Query Module
**Location**: `/backend/src/graph_query.py`

**Functionality**:
- Neo4j driver management
- Cypher query execution
- Graph visualization data preparation
- Schema extraction

**Key Functions**:

- `get_graphDB_driver()`: Creates Neo4j driver instance with credentials
- `get_graph_results()`: Retrieves graph data for visualization
  - Fetches documents, chunks, entities, relationships
  - Includes community information
  - Returns nodes and relationships for frontend

- `get_chunktext_results()`: Retrieves chunk text and metadata
- `visualize_schema()`: Extracts database schema
  - Node labels and their counts
  - Relationship types
  - Properties for each label

### 8. **post_processing.py** - Graph Post-Processing
**Location**: `/backend/src/post_processing.py`

**Functionality**:
- Creates vector and fulltext indexes
- Generates entity embeddings
- Schema consolidation
- Graph cleanup and optimization

**Key Functions**:

#### Index Creation
- `create_vector_index()`: Creates vector indexes for embeddings
  - Chunk embeddings for semantic search
  - Configurable dimensions (default 384 for MiniLM)
  - Cosine similarity function

- `create_fulltext()`: Creates fulltext indexes
  - Entity search index
  - Chunk keyword search
  - Community summary search

#### Entity Embeddings
- `create_entity_embedding()`: Generates embeddings for entities
  - Uses entity ID and description
  - Stores embeddings on entity nodes
  - Enables entity-based semantic search

#### Schema Consolidation
- `graph_schema_consolidation()`: Cleans and consolidates graph
  - Uses LLM to identify duplicate entities
  - Merges similar entities
  - Consolidates relationships
  - Improves graph quality

### 9. **communities.py** - Community Detection
**Location**: `/backend/src/communities.py`

**Functionality**:
- Detects communities in knowledge graphs
- Hierarchical community structure
- Community summarization using LLMs
- Ranking and weighting

**Community Detection Algorithm**:
1. Creates graph projection excluding Chunks and Documents
2. Uses Leiden algorithm for community detection
3. Creates hierarchical levels (up to 3 levels)
4. Generates community summaries using LLM
5. Calculates community ranks and weights

**Key Components**:

- `create_communities()`: Main community creation pipeline
  - Creates `__Community__` nodes
  - Establishes `IN_COMMUNITY` relationships
  - Creates `PARENT_COMMUNITY` hierarchies
  - Generates summaries for communities

- Community summarization:
  - Uses LLM to summarize community contents
  - Includes entity descriptions and relationships
  - Creates human-readable summaries

**Configuration**:
- `MAX_COMMUNITY_LEVELS`: 3 (configurable)
- `MIN_COMMUNITY_SIZE`: 1 (minimum entities per community)
- Default model: GPT-4o


### 10. **chunkid_entities.py** - Chunk-Entity Mapping
**Location**: `/backend/src/chunkid_entities.py`

**Functionality**:
- Maps chunks to their extracted entities
- Retrieves entities for specific chunks
- Supports chat context enrichment

**Key Functions**:
- `get_entities_from_chunkids()`: Retrieves all entities from specified chunks
  - Returns entity details (id, type, description)
  - Used for displaying context in chat responses

### 11. **make_relationships.py** - Relationship Creation
**Location**: `/backend/src/make_relationships.py`

**Functionality**:
- Creates relationships between chunks
- Implements chunk similarity detection
- Manages chunk ordering

**Relationship Types**:
- `FIRST_CHUNK`: Links document to first chunk
- `PART_OF`: Links chunks to their document
- `NEXT_CHUNK`: Sequential ordering of chunks
- `SIMILAR`: Connects semantically similar chunks

**Similarity Detection**:
- Uses embedding similarity
- Configurable threshold (KNN_MIN_SCORE)
- Prevents duplicate similar relationships

### 12. **neighbours.py** - Graph Neighbor Operations
**Location**: `/backend/src/neighbours.py`

**Functionality**:
- Retrieves neighboring nodes in graph
- Supports graph expansion in UI
- Provides relationship context

### 13. **ragas_eval.py** - RAG Evaluation
**Location**: `/backend/src/ragas_eval.py`

**Functionality**:
- Evaluates RAG (Retrieval-Augmented Generation) performance
- Uses RAGAS framework
- Metrics: faithfulness, answer relevancy, context precision

**Evaluation Metrics**:
- Answer relevancy: How relevant is the answer to the question?
- Faithfulness: Is the answer grounded in the context?
- Context precision: Quality of retrieved context

### 14. **Document Source Modules**
**Location**: `/backend/src/document_sources/`

These modules handle different input sources:

#### local_file.py
- Loads documents from local filesystem
- Supports PDF, TXT, DOCX formats
- Uses LangChain document loaders

#### gcs_bucket.py
- Google Cloud Storage integration
- OAuth authentication
- File metadata extraction
- Upload and download operations

#### s3_bucket.py
- AWS S3 bucket integration
- Access key authentication
- File listing and retrieval

#### youtube.py
- YouTube video transcript extraction
- Timestamp preservation
- Uses youtube-transcript-api
- Handles multiple languages

#### wikipedia.py
- Wikipedia article extraction
- Uses WikipediaLoader from LangChain
- Fetches article content and metadata

#### web_pages.py
- Web scraping functionality
- Uses WebBaseLoader
- Extracts main content from web pages

### 15. **Entity Models**
**Location**: `/backend/src/entities/`

#### source_node.py
- Defines `sourceNode` class
- Represents document metadata
- Properties: file_name, file_size, file_type, status, url, model, etc.

#### user_credential.py
- Manages user credentials for Neo4j
- Database connection information

### 16. **Shared Utilities**
**Location**: `/backend/src/shared/`

#### constants.py
- Defines constants and Cypher queries
- Graph query templates
- Chunk queries
- Node and relationship patterns
- LLM prompts for extraction

#### common_fn.py
- Utility functions used across modules
- Embedding model loading
- File operations
- Hashing functions
- Query execution helpers

#### schema_extraction.py
- Extracts entity schema from text
- Uses LLM to identify entity types and relationships
- Configurable schema definitions

#### llm_graph_builder_exception.py
- Custom exception classes
- Error handling and reporting

### 17. **logger.py** - Logging Module
**Location**: `/backend/src/logger.py`

**Functionality**:
- Custom logging configuration
- Structured logging
- Log level management


---

## Frontend Components

The frontend is built with React and TypeScript, providing an interactive UI for graph building and exploration.

### Application Structure

**Location**: `/frontend/src/`

### 1. **App.tsx** - Application Root
**Functionality**:
- Defines application routes
- Authentication guard integration
- Three main routes:
  - `/`: Main application (with optional auth)
  - `/readonly`: Read-only mode
  - `/chat-only`: Standalone chat interface

### 2. **Home.tsx** - Home Component
**Functionality**:
- Main landing page
- Theme wrapper
- Google OAuth provider (for GCS integration)
- Error boundary
- Toaster for notifications
- Renders QuickStarter component

### 3. **QuickStarter.tsx** - Main Application Container
**Location**: `/frontend/src/components/QuickStarter.tsx`

**Functionality**:
- Main application layout
- Coordinates all major components
- State management for:
  - Selected files
  - Graph view state
  - Chat interface state
  - Upload progress

### Core Component Categories

### 4. **Layout Components**
**Location**: `/frontend/src/components/Layout/`

#### Header.tsx
- Application header with branding
- Navigation elements
- User information display

#### SideNav.tsx
- Side navigation panel
- Settings and configuration access
- Quick action buttons

#### PageLayout.tsx
- Overall page structure
- Responsive layout
- Content area management

#### DrawerChatbot.tsx
- Drawer for chat interface
- Slide-in/out functionality
- Chat history display

#### DrawerDropzone.tsx
- File upload drawer
- Drag-and-drop interface
- Upload progress tracking

#### AlertIcon.tsx
- Alert and notification icons
- Status indicators

### 5. **Data Source Components**
**Location**: `/frontend/src/components/DataSources/`

These components handle different data sources:

#### Local File Upload
**Location**: `DataSources/Local/`
- Local file selection
- Drag-and-drop support
- File validation
- Chunked upload for large files

#### AWS S3 Integration
**Location**: `DataSources/AWS/`
- S3 bucket configuration
- Access key input
- Bucket folder selection
- File listing from S3

#### Google Cloud Storage
**Location**: `DataSources/GCS/`
- OAuth authentication
- GCS bucket selection
- Project ID configuration
- File picker interface

#### Web Sources
**Location**: `DataSources/Web/`
- URL input for web pages
- Wikipedia query input
- YouTube URL input
- URL validation

### 6. **File Management Components**

#### FileTable.tsx
**Location**: `/frontend/src/components/FileTable.tsx`

**Functionality**:
- Displays uploaded files in table format
- Columns: File name, Size, Source, Status, Model, Actions
- Status indicators: New, Processing, Completed, Failed, Cancelled
- Actions:
  - Generate Graph: Start entity extraction
  - View: Preview generated graph
  - Delete: Remove file and entities
  - Retry: Reprocess failed files
- Bulk operations support
- Sorting and filtering
- Pagination

#### Content.tsx
**Functionality**:
- Content display area
- File list management
- Status updates
- Real-time processing feedback

### 7. **Graph Visualization Components**
**Location**: `/frontend/src/components/Graph/`

#### GraphViewModal.tsx
**Functionality**:
- Modal for graph visualization
- Neo4j Bloom integration
- Full-screen graph view
- Interactive graph exploration

#### GraphViewButton.tsx
**Functionality**:
- Button to open graph view
- Enabled when files are selected
- Links to Neo4j Bloom

#### GraphPropertiesPanel.tsx
**Functionality**:
- Displays node and relationship properties
- Property inspector
- Metadata viewing

#### GraphPropertiesTable.tsx
**Functionality**:
- Table view of graph properties
- Property names and values
- Type information

#### SchemaViz.tsx
**Functionality**:
- Visualizes graph schema
- Shows entity types and relationships
- Node and relationship counts

#### SchemaDropdown.tsx
**Functionality**:
- Dropdown for schema selection
- Custom schema configuration
- Predefined schema templates

#### CheckboxSelection.tsx
**Functionality**:
- Checkbox for selecting multiple files
- Bulk action enablement

#### LegendsChip.tsx
**Functionality**:
- Legend display for graph visualization
- Color coding for node types
- Relationship type indicators

#### ResizePanel.tsx
**Functionality**:
- Resizable panel for graph view
- Drag to resize
- Responsive layout

#### ResultOverview.tsx
**Functionality**:
- Overview of extraction results
- Statistics display:
  - Number of entities extracted
  - Number of relationships created
  - Chunk counts
  - Processing time


### 8. **ChatBot Components**
**Location**: `/frontend/src/components/ChatBot/`

#### Chatbot.tsx
**Functionality**:
- Main chat interface
- Message input and display
- Real-time streaming responses
- Chat history management
- Context-aware conversations

#### ChatOnlyComponent.tsx
**Functionality**:
- Standalone chat interface
- Accessible via `/chat-only` route
- Full-screen chat experience
- No file upload interface

#### ChatModeToggle.tsx
**Functionality**:
- Toggle between chat modes
- Vector, Graph, Hybrid, etc.
- Mode descriptions and help

#### ChatModesSwitch.tsx
**Functionality**:
- Switch component for chat modes
- Visual mode selection
- Mode-specific settings

#### ChatInfoModal.tsx
**Functionality**:
- Modal displaying chat response metadata
- Source attribution
- Confidence scores
- Retrieved context

#### SourcesInfo.tsx
**Functionality**:
- Displays source documents for answers
- Document links
- Relevance scores

#### ChunkInfo.tsx
**Functionality**:
- Shows chunk information
- Chunk text display
- Chunk metadata (page number, timestamps)

#### EntitiesInfo.tsx
**Functionality**:
- Lists entities used in answer generation
- Entity types and descriptions
- Entity relationships

#### CommunitiesInfo.tsx
**Functionality**:
- Displays community information
- Community summaries
- Hierarchical structure

#### MetricsTab.tsx
**Functionality**:
- Metrics and analytics tab
- Response quality metrics
- Token usage
- Processing time

#### MetricsCheckbox.tsx
**Functionality**:
- Checkbox for enabling metrics display
- Toggle detailed metrics

#### MultiModeMetrics.tsx
**Functionality**:
- Metrics for multi-mode retrieval
- Comparison across modes
- Performance indicators

#### NotAvailableMetric.tsx
**Functionality**:
- Placeholder for unavailable metrics
- Graceful degradation

#### CommonChatActions.tsx
**Functionality**:
- Common chat actions (copy, share, feedback)
- Message actions
- History management

#### ExpandedChatButtonContainer.tsx
**Functionality**:
- Container for expanded chat buttons
- Quick actions
- Settings access

### 9. **Popup Components**
**Location**: `/frontend/src/components/Popups/`

#### GraphEnhancementDialog
**Functionality**:
- Dialog for graph enhancement settings
- Entity extraction configuration
- Schema customization
- Additional instructions for LLM

**Sub-components**:
- `AdditionalInstructions/`: Custom instructions input
- `EntityExtraction/`: Entity type configuration
- `ImporterInput.tsx`: Import/export schema

#### Settings Dialogs
- LLM model selection
- Chunk size configuration
- Embedding model settings
- Database connection settings

### 10. **Authentication Components**
**Location**: `/frontend/src/components/Auth/`

#### Auth.tsx
**Functionality**:
- Authentication guard component
- OAuth integration (Auth0)
- Login/logout functionality
- Session management
- Protected route wrapper

### 11. **Login Component**
**Location**: `/frontend/src/components/Login/`

**Functionality**:
- Login form
- Neo4j credentials input
- Connection testing
- Credential file upload (.txt with connection details)

### 12. **UI Components**
**Location**: `/frontend/src/components/`

#### Dropdown.tsx
- Reusable dropdown component
- Model selection
- Source selection
- Settings dropdowns

#### BreakDownPopOver.tsx
- Popover for detailed breakdowns
- Metric explanations
- Help text

### Frontend Services

**Location**: `/frontend/src/services/`

These modules handle API communication:

#### API Services
- File upload service
- Document extraction service
- Chat service (WebSocket for streaming)
- Graph query service
- Status polling service

### Context and State Management

**Location**: `/frontend/src/context/`

#### ThemeWrapper
**Functionality**:
- Theme management (light/dark mode)
- Neo4j NDL theme integration
- User preference persistence

### Hooks

**Location**: `/frontend/src/hooks/`

Custom React hooks for:
- File upload management
- Chat state management
- Graph data fetching
- Polling for status updates

### Utilities

**Location**: `/frontend/src/utils/`

#### Constants.ts
**Functionality**:
- Application constants
- API endpoints
- Configuration values
- Environment variable access

### Types

**Location**: `/frontend/src/types.ts`

TypeScript type definitions for:
- File objects
- Graph nodes and relationships
- Chat messages
- API responses
- User credentials


---

## Data Flow

### 1. Document Upload and Processing Flow

```
User Uploads File
       ↓
Frontend validates file
       ↓
Chunked upload to backend (/upload endpoint)
       ↓
Backend saves chunks temporarily
       ↓
Merge chunks (/merge endpoint)
       ↓
Create Document node in Neo4j (status: New)
       ↓
User triggers "Generate Graph" (/extract endpoint)
       ↓
Backend processing:
  1. Load document from source
  2. Split into chunks
  3. Create Chunk nodes
  4. For each chunk:
     a. Send to LLM for entity extraction
     b. Create entity nodes
     c. Create relationships
     d. Update progress
  5. Create chunk relationships (NEXT_CHUNK, SIMILAR)
  6. Post-processing (embeddings, indexes)
       ↓
Update Document status to "Completed"
       ↓
Frontend displays results and graph
```

### 2. Chat/QA Flow

```
User enters question
       ↓
Frontend sends to /chat_bot endpoint
       ↓
Backend processing:
  1. Load chat history for session
  2. Transform question (if needed)
  3. Based on chat mode:
     - Vector: Retrieve similar chunks via embedding
     - Graph: Generate Cypher query and execute
     - Hybrid: Combine multiple strategies
  4. Retrieve relevant context
  5. Send to LLM with context
  6. Stream response back
       ↓
Frontend displays answer with sources
       ↓
Update chat history
```

### 3. Graph Visualization Flow

```
User clicks "View" or "Preview Graph"
       ↓
Frontend calls /graph_results endpoint
       ↓
Backend executes complex Cypher query:
  1. Fetch Document nodes
  2. Fetch related Chunks
  3. Fetch Entities from chunks
  4. Fetch Relationships between entities
  5. Fetch Communities (if enabled)
       ↓
Return nodes and relationships as JSON
       ↓
Frontend opens Neo4j Bloom with data
       ↓
User explores interactive graph
```

### 4. Community Detection Flow

```
User triggers community creation
       ↓
Backend creates graph projection
       ↓
Run Leiden community detection algorithm
       ↓
Create hierarchical community structure:
  - Level 0: Base communities
  - Level 1: Parent communities
  - Level 2: Top-level communities
       ↓
For each community:
  1. Collect member entities
  2. Generate summary using LLM
  3. Calculate rank and weight
       ↓
Create Community nodes and relationships
       ↓
Update fulltext index for community search
```

---

## Key Features

### 1. Multi-Source Data Ingestion

The application supports various data sources:

- **Local Files**: PDF, TXT, DOCX via file upload
- **Cloud Storage**:
  - Google Cloud Storage (with OAuth)
  - AWS S3 (with access keys)
- **Web Sources**:
  - Any web page URL
  - Wikipedia articles (by search query)
  - YouTube videos (with transcript extraction)

### 2. LLM-Powered Entity Extraction

- Uses multiple LLM providers for flexibility
- Extracts:
  - **Entities**: People, organizations, locations, concepts, etc.
  - **Relationships**: Typed connections between entities
  - **Properties**: Attributes of entities and relationships
- Schema-aware extraction with custom entity types
- Additional instructions for domain-specific extraction

### 3. Knowledge Graph Construction

- **Graph Structure**:
  - `Document` nodes: Represent source files
  - `Chunk` nodes: Text segments from documents
  - `Entity` nodes: Extracted entities
  - `__Community__` nodes: Detected communities
  
- **Relationships**:
  - `PART_OF`: Chunk → Document
  - `FIRST_CHUNK`: Document → First Chunk
  - `NEXT_CHUNK`: Chunk → Next Chunk (sequential)
  - `SIMILAR`: Chunk → Chunk (semantic similarity)
  - `HAS_ENTITY`: Chunk → Entity
  - Domain relationships: Between entities (e.g., WORKS_FOR, LOCATED_IN)
  - `IN_COMMUNITY`: Entity → Community
  - `PARENT_COMMUNITY`: Community → Community (hierarchical)

### 4. Advanced Search and Retrieval

Multiple retrieval strategies:

1. **Vector Search**: Embedding-based semantic search
2. **Graph Search**: Cypher query generation by LLM
3. **Fulltext Search**: Keyword-based using Neo4j indexes
4. **Hybrid Search**: Combination of above
5. **Entity Vector**: Entity-focused semantic search
6. **Global Vector**: Community summary search

### 5. Conversational QA

- Chat interface over knowledge graphs
- Context-aware responses with history
- Source attribution (which documents/chunks)
- Streaming responses for better UX
- Multiple chat modes for different use cases

### 6. Community Detection

- Hierarchical community structure
- Leiden algorithm for detection
- LLM-generated summaries
- Community ranking and weighting
- Enables high-level insights from large graphs

### 7. Graph Visualization

- Integration with Neo4j Bloom
- Interactive graph exploration
- Schema visualization
- Property inspection
- Export capabilities

### 8. Post-Processing Capabilities

- **Duplicate Detection**: Identifies and merges similar entities
- **Entity Embeddings**: Enables semantic search on entities
- **Index Creation**: Vector, fulltext, and graph indexes
- **Schema Consolidation**: Cleans and optimizes graph structure

### 9. Monitoring and Metrics

- Processing status tracking
- Token usage monitoring
- Response quality metrics (RAGAS)
- Performance analytics
- Error logging and reporting

### 10. Scalability Features

- Chunked file upload for large files
- Parallel chunk processing
- Configurable chunk sizes
- Resume capability (process from last position)
- Cancellation support


---

## Deployment

### Deployment Options

#### 1. Docker Compose (Recommended for Local Development)

**File**: `/docker-compose.yml`

Services:
- `backend`: FastAPI application
- `frontend`: React application (Nginx)
- `database`: Neo4j (optional, use external for production)

Steps:
```bash
# Set environment variables in .env file
cp backend/example.env backend/.env
cp frontend/example.env frontend/.env

# Start services
docker-compose up -d

# Access application
# Frontend: http://localhost:80
# Backend: http://localhost:8000
# Neo4j Browser: http://localhost:7474
```

#### 2. Separate Backend and Frontend

**Backend**:
```bash
cd backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
uvicorn score:app --reload --host 0.0.0.0 --port 8000
```

**Frontend**:
```bash
cd frontend
yarn install
yarn run dev
```

#### 3. Cloud Deployment (Google Cloud Run)

**Frontend**:
```bash
gcloud run deploy llm-graph-builder-frontend \
  --source ./frontend \
  --region us-central1 \
  --allow-unauthenticated \
  --set-env-vars VITE_BACKEND_API_URL=<backend-url>
```

**Backend**:
```bash
gcloud run deploy llm-graph-builder-backend \
  --source ./backend \
  --region us-central1 \
  --allow-unauthenticated \
  --set-env-vars OPENAI_API_KEY=<key> \
  --set-env-vars NEO4J_URI=<uri> \
  --set-env-vars NEO4J_USERNAME=<username> \
  --set-env-vars NEO4J_PASSWORD=<password>
```

#### 4. Kubernetes Deployment

Create Kubernetes manifests for:
- Backend deployment and service
- Frontend deployment and service
- Ingress for routing
- ConfigMaps for configuration
- Secrets for sensitive data

### Infrastructure Requirements

**Minimum Requirements**:
- CPU: 2 cores
- RAM: 4 GB
- Storage: 20 GB
- Neo4j: Version 5.23+ with APOC plugin

**Recommended for Production**:
- CPU: 4-8 cores
- RAM: 16-32 GB
- Storage: 100+ GB SSD
- Neo4j: Dedicated instance with adequate resources
- Load balancer for high availability

---

## Configuration

### Backend Environment Variables

**Required**:
- `OPENAI_API_KEY`: OpenAI API key
- `DIFFBOT_API_KEY`: Diffbot API key (if using Diffbot)
- `NEO4J_URI`: Neo4j connection URI (e.g., neo4j://localhost:7687)
- `NEO4J_USERNAME`: Neo4j username (default: neo4j)
- `NEO4J_PASSWORD`: Neo4j password

**Optional LLM Configurations**:
- `LLM_MODEL_CONFIG_gemini_<model>`: Gemini model configuration
- `LLM_MODEL_CONFIG_azure_<model>`: Azure OpenAI configuration
- `LLM_MODEL_CONFIG_anthropic_<model>`: Anthropic configuration
- `LLM_MODEL_CONFIG_groq_<model>`: Groq configuration
- `LLM_MODEL_CONFIG_bedrock_<model>`: AWS Bedrock configuration
- `LLM_MODEL_CONFIG_ollama_<model>`: Ollama configuration
- `LLM_MODEL_CONFIG_fireworks_<model>`: Fireworks AI configuration

**Processing Configuration**:
- `EMBEDDING_MODEL`: Embedding model (default: all-MiniLM-L6-v2)
- `IS_EMBEDDING`: Enable embeddings (default: true)
- `ENTITY_EMBEDDING`: Enable entity embeddings (default: false)
- `MAX_TOKEN_CHUNK_SIZE`: Maximum tokens per chunk (default: 10000)
- `NUMBER_OF_CHUNKS_TO_COMBINE`: Chunks to combine (default: 5)
- `UPDATE_GRAPH_CHUNKS_PROCESSED`: Update frequency (default: 20)
- `KNN_MIN_SCORE`: Similarity threshold (default: 0.94)
- `DUPLICATE_TEXT_DISTANCE`: Distance for duplicate detection (default: 5)
- `DUPLICATE_SCORE_VALUE`: Score for duplicate matching (default: 0.97)

**Storage Configuration**:
- `GCS_FILE_CACHE`: Use GCS for file storage (default: false)
- `BUCKET`: GCS bucket name for uploads

**Monitoring**:
- `LANGCHAIN_API_KEY`: LangSmith API key for tracing
- `LANGCHAIN_PROJECT`: LangSmith project name
- `LANGCHAIN_TRACING_V2`: Enable tracing (default: true)
- `GCP_LOG_METRICS_ENABLED`: Enable GCP logging (default: false)

**Advanced**:
- `NEO4J_USER_AGENT`: Custom user agent (default: llm-graph-builder)
- `ENABLE_USER_AGENT`: Enable user agent tracking (default: true)
- `GRAPH_CLEANUP_MODEL`: Model for graph cleanup (default: 0.97)
- `YOUTUBE_TRANSCRIPT_PROXY`: Proxy for YouTube transcripts
- `RAGAS_EMBEDDING_MODEL`: Embedding model for RAGAS (default: openai)

### Frontend Environment Variables

**Required**:
- `VITE_BACKEND_API_URL`: Backend API URL (default: http://localhost:8000)
- `VITE_BLOOM_URL`: Neo4j Bloom URL template
- `VITE_REACT_APP_SOURCES`: Enabled sources (e.g., local,youtube,wiki,s3,gcs,web)
- `VITE_CHAT_MODES`: Enabled chat modes
- `VITE_LLM_MODELS`: Available LLM models
- `VITE_ENV`: Environment (DEV or PROD)

**Optional**:
- `VITE_GOOGLE_CLIENT_ID`: Google OAuth client ID (for GCS)
- `VITE_LLM_MODELS_PROD`: Production-only models
- `VITE_TIME_PER_PAGE`: Processing time estimate (default: 50)
- `VITE_CHUNK_SIZE`: Upload chunk size (default: 5242880)
- `VITE_AUTH0_CLIENT_ID`: Auth0 client ID
- `VITE_AUTH0_DOMAIN`: Auth0 domain
- `VITE_SKIP_AUTH`: Skip authentication (default: true)
- `VITE_CHUNK_OVERLAP`: Chunk overlap size (default: 20)
- `VITE_TOKENS_PER_CHUNK`: Tokens per chunk (default: 100)
- `VITE_CHUNK_TO_COMBINE`: Chunks to combine (default: 1)

### Neo4j Configuration

**Database Requirements**:
- Version: 5.23 or later
- APOC plugin: Required for graph operations
- GDS plugin: Required for community detection (optional)

**Indexes**:
Created automatically by the application:
- Vector index on `Chunk.embedding`
- Fulltext index on entities
- Fulltext index on chunks (keyword search)
- Fulltext index on communities

**Constraints**:
- `Document.fileName` must be unique
- `__Community__.id` must be unique
- `Chunk.id` must be unique

**Memory Recommendations**:
- Heap: 4-8 GB minimum
- Page cache: 2-4 GB minimum
- Adjust based on graph size


---

## Security Considerations

### Input Validation
- Filename sanitization to prevent path traversal
- File type validation
- Size limits on uploads
- URL validation for web sources

### Authentication
- Optional Auth0 integration
- Neo4j credential management
- Session management with secure cookies
- OAuth for cloud storage access

### Data Protection
- Environment variable management for secrets
- No hardcoded credentials
- Secure file storage
- CORS configuration

### API Security
- CORS middleware with allowed origins
- Input sanitization
- Error message sanitization (no stack traces to clients)
- Rate limiting (recommended to add)

---

## Performance Optimization

### Backend Optimizations
- Batch processing of chunks
- Parallel entity extraction
- Connection pooling for Neo4j
- Caching of embeddings
- Lazy loading of documents

### Frontend Optimizations
- Code splitting
- Lazy loading of components
- Debouncing of search inputs
- Pagination for large lists
- Memoization of expensive computations

### Database Optimizations
- Proper indexing strategy
- Query optimization
- Batch operations
- Connection pooling
- Regular maintenance (APOC procedures)

### Chunking Strategy
- Configurable chunk sizes
- Overlap for context preservation
- Token-based splitting
- Metadata preservation

---

## Troubleshooting

### Common Issues

1. **Neo4j Connection Failed**
   - Check URI, username, password
   - Verify Neo4j is running
   - Check network connectivity
   - Verify APOC plugin is installed

2. **LLM API Errors**
   - Verify API keys are correct
   - Check API quota/rate limits
   - Verify model names are correct
   - Check network connectivity

3. **Upload Failures**
   - Check file size limits
   - Verify disk space
   - Check file permissions
   - Verify upload directory exists

4. **Slow Processing**
   - Reduce chunk size
   - Use faster LLM models
   - Increase concurrent workers
   - Optimize Neo4j configuration

5. **Memory Issues**
   - Increase heap size for Neo4j
   - Reduce batch sizes
   - Enable GCS file cache
   - Garbage collection tuning

---

## Development Workflow

### Adding a New LLM Provider

1. Update `llm.py`:
   - Add model initialization logic
   - Handle API key configuration
   - Set temperature and parameters

2. Update frontend constants:
   - Add model name to `VITE_LLM_MODELS`
   - Update UI dropdown

3. Add environment variable:
   - `LLM_MODEL_CONFIG_<provider>_<model>`

4. Test extraction and chat

### Adding a New Data Source

1. Create module in `document_sources/`:
   - Implement data fetching
   - Return LangChain Documents

2. Update `main.py`:
   - Add source node creation function
   - Add extraction handler

3. Update frontend:
   - Create UI component in `DataSources/`
   - Add to sources configuration
   - Update API calls

4. Test end-to-end flow

### Adding a New Chat Mode

1. Update `QA_integration.py`:
   - Implement retrieval logic
   - Create LangChain chain

2. Update frontend:
   - Add mode to `VITE_CHAT_MODES`
   - Update ChatModeToggle component

3. Test retrieval and response quality

---

## Testing

### Backend Testing
- Unit tests for individual modules
- Integration tests for API endpoints
- Performance tests for large files
- LLM integration tests

### Frontend Testing
- Component tests with React Testing Library
- E2E tests with Playwright/Cypress
- Accessibility testing
- Browser compatibility testing

### Database Testing
- Cypher query validation
- Index performance testing
- Data integrity checks
- Backup and restore testing

---

## Monitoring and Observability

### Logging
- Structured logging with context
- Log levels: DEBUG, INFO, WARNING, ERROR
- Request/response logging
- Error stack traces

### Metrics
- Processing time per document
- Token usage per LLM call
- Database query performance
- API response times
- Error rates

### Tracing
- LangSmith integration for LLM tracing
- Request tracing across services
- Database query tracing
- Performance bottleneck identification

---

## Future Enhancements

### Planned Features
- Real-time collaboration
- Advanced schema visualization
- Graph analytics dashboard
- Automated schema learning
- Multi-language support
- Streaming ingestion
- GraphRAG improvements
- Fine-tuned models for entity extraction

### Scalability Improvements
- Distributed processing
- Message queue integration
- Caching layer (Redis)
- CDN for frontend
- Database sharding

---

## Glossary

**Chunk**: A segment of text from a document, typically 100-500 tokens.

**Entity**: A real-world object or concept extracted from text (person, place, organization, etc.).

**Relationship**: A typed connection between two entities.

**Embedding**: A vector representation of text in high-dimensional space.

**Knowledge Graph**: A network of entities and their relationships.

**Community**: A group of densely connected entities in the graph.

**Cypher**: Neo4j's graph query language.

**RAG**: Retrieval-Augmented Generation, combining retrieval with LLM generation.

**Vector Search**: Finding similar items using embedding similarity.

**Fulltext Search**: Keyword-based search using text indexes.

**Graph Traversal**: Navigating relationships in a graph database.

**Schema**: The structure defining entity types and relationship types.

---

## Conclusion

The LLM Graph Builder is a comprehensive application that transforms unstructured data into structured, queryable knowledge graphs. It combines state-of-the-art LLMs with graph database technology to enable powerful semantic search and question-answering capabilities. The modular architecture allows for easy extension and customization, while the multi-modal approach to data ingestion and retrieval ensures flexibility for various use cases.

For questions, issues, or contributions, please visit the [GitHub repository](https://github.com/neo4j-labs/llm-graph-builder).

---

*Last Updated: 2025-10-29*
