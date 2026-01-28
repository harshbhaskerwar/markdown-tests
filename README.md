# Section-Based Streaming Implementation Guide
## Python Backend + Flutter Frontend for Long-Form AI Content Generation

---

## Table of Contents

1. [System Architecture Overview](#1-system-architecture-overview)
2. [Python Backend Implementation](#2-python-backend-implementation)
3. [Flutter Frontend Integration](#3-flutter-frontend-integration)
4. [Production Considerations](#4-production-considerations)
5. [Deployment & Scaling](#5-deployment--scaling)

---

## 1. System Architecture Overview

### 1.1 High-Level Architecture

```
[Flutter Client]
      ↓ WebSocket/SSE
[Python Backend - FastAPI/Flask]
      ↓ API Calls
[LLM Provider - OpenAI/Anthropic]
      ↓ Token Streaming
[Database - PostgreSQL]
      ↓ State Persistence
[Redis - Session & Queue Management]
      ↓ Background Processing
[Celery Workers - Recovery & Retry]
```

### 1.2 Core Components

**Backend Responsibilities:**
- Manage section orchestration
- Handle streaming connections
- Persist generation state
- Implement retry/recovery logic
- Rate limiting & queue management

**Frontend Responsibilities:**
- Maintain WebSocket/SSE connection
- Display streaming text in real-time
- Handle reconnection logic
- Show progress indicators
- Cache received content

---

## 2. Python Backend Implementation

### 2.1 Technology Stack

```python
# requirements.txt
fastapi==0.109.0
uvicorn[standard]==0.27.0
python-multipart==0.0.6
sqlalchemy==2.0.25
psycopg2-binary==2.9.9
redis==5.0.1
celery==5.3.6
openai==1.10.0  # or anthropic==0.18.0
pydantic==2.6.0
sse-starlette==1.8.2
python-dotenv==1.0.0
tenacity==8.2.3
```

### 2.2 Project Structure

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py                 # FastAPI application
│   ├── config.py               # Configuration management
│   ├── models/
│   │   ├── __init__.py
│   │   ├── generation.py       # SQLAlchemy models
│   │   └── schemas.py          # Pydantic schemas
│   ├── services/
│   │   ├── __init__.py
│   │   ├── llm_service.py      # LLM API integration
│   │   ├── orchestrator.py     # Section management
│   │   └── stream_service.py   # Streaming logic
│   ├── api/
│   │   ├── __init__.py
│   │   ├── routes.py           # API endpoints
│   │   └── websocket.py        # WebSocket handlers
│   ├── tasks/
│   │   ├── __init__.py
│   │   └── celery_tasks.py     # Background jobs
│   └── utils/
│       ├── __init__.py
│       ├── redis_client.py
│       └── database.py
├── alembic/                    # Database migrations
├── tests/
└── .env
```

### 2.3 Database Models

```python
# app/models/generation.py
from sqlalchemy import Column, String, Integer, DateTime, JSON, Enum, Text
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime
import enum

Base = declarative_base()

class GenerationStatus(str, enum.Enum):
    PENDING = "pending"
    IN_PROGRESS = "in_progress"
    COMPLETED = "completed"
    FAILED = "failed"
    PAUSED = "paused"

class SectionStatus(str, enum.Enum):
    PENDING = "pending"
    GENERATING = "generating"
    COMPLETED = "completed"
    FAILED = "failed"

class Generation(Base):
    __tablename__ = "generations"
    
    id = Column(String, primary_key=True)
    user_id = Column(String, nullable=False, index=True)
    project_type = Column(String, nullable=False)  # e.g., "screenplay"
    status = Column(Enum(GenerationStatus), default=GenerationStatus.PENDING)
    metadata = Column(JSON, default={})  # User inputs, preferences
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    completed_at = Column(DateTime, nullable=True)
    error_message = Column(Text, nullable=True)
    
class Section(Base):
    __tablename__ = "sections"
    
    id = Column(String, primary_key=True)
    generation_id = Column(String, nullable=False, index=True)
    section_name = Column(String, nullable=False)  # e.g., "title_page", "act_1"
    section_order = Column(Integer, nullable=False)
    status = Column(Enum(SectionStatus), default=SectionStatus.PENDING)
    content = Column(Text, default="")
    prompt_used = Column(Text)
    tokens_used = Column(Integer, default=0)
    model_name = Column(String)
    started_at = Column(DateTime, nullable=True)
    completed_at = Column(DateTime, nullable=True)
    error_message = Column(Text, nullable=True)
    retry_count = Column(Integer, default=0)
```

### 2.4 Pydantic Schemas

```python
# app/models/schemas.py
from pydantic import BaseModel, Field
from typing import Optional, Dict, Any, List
from datetime import datetime

class GenerationRequest(BaseModel):
    user_id: str
    project_type: str
    metadata: Dict[str, Any] = Field(default_factory=dict)
    
    class Config:
        json_schema_extra = {
            "example": {
                "user_id": "user_123",
                "project_type": "screenplay",
                "metadata": {
                    "title": "The Last Stand",
                    "genre": "Action",
                    "tone": "Dark",
                    "length": "feature"
                }
            }
        }

class GenerationResponse(BaseModel):
    generation_id: str
    status: str
    websocket_url: str
    
class SectionUpdate(BaseModel):
    section_name: str
    status: str
    content_chunk: Optional[str] = None
    is_complete: bool = False
    tokens_used: Optional[int] = None
    error: Optional[str] = None

class StreamMessage(BaseModel):
    type: str  # "section_start", "content", "section_complete", "error", "complete"
    generation_id: str
    section_name: Optional[str] = None
    content: Optional[str] = None
    metadata: Optional[Dict[str, Any]] = None
```

### 2.5 Configuration Management

```python
# app/config.py
from pydantic_settings import BaseSettings
from functools import lru_cache

class Settings(BaseSettings):
    # Application
    APP_NAME: str = "Section-Based Streaming API"
    DEBUG: bool = False
    
    # Database
    DATABASE_URL: str
    
    # Redis
    REDIS_URL: str = "redis://localhost:6379/0"
    
    # LLM Provider
    OPENAI_API_KEY: str
    OPENAI_MODEL: str = "gpt-4-turbo-preview"
    MAX_TOKENS_PER_SECTION: int = 4000
    
    # Streaming
    STREAM_CHUNK_SIZE: int = 50  # Characters to buffer before sending
    HEARTBEAT_INTERVAL: int = 30  # Seconds
    
    # Retry Logic
    MAX_RETRIES: int = 3
    RETRY_DELAY: int = 5  # Seconds
    
    # Rate Limiting
    RATE_LIMIT_PER_USER: int = 5  # Concurrent generations
    
    class Config:
        env_file = ".env"

@lru_cache()
def get_settings():
    return Settings()
```

### 2.6 LLM Service Implementation

```python
# app/services/llm_service.py
from openai import AsyncOpenAI
from typing import AsyncIterator, Dict, Any
from tenacity import retry, stop_after_attempt, wait_exponential
import logging

logger = logging.getLogger(__name__)

class LLMService:
    def __init__(self, api_key: str, model: str):
        self.client = AsyncOpenAI(api_key=api_key)
        self.model = model
    
    @retry(
        stop=stop_after_attempt(3),
        wait=wait_exponential(multiplier=1, min=4, max=10)
    )
    async def generate_section_stream(
        self,
        system_prompt: str,
        user_prompt: str,
        max_tokens: int = 4000,
        temperature: float = 0.7
    ) -> AsyncIterator[str]:
        """
        Streams a single section from the LLM.
        Yields content chunks as they arrive.
        """
        try:
            response = await self.client.chat.completions.create(
                model=self.model,
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                max_tokens=max_tokens,
                temperature=temperature,
                stream=True
            )
            
            async for chunk in response:
                if chunk.choices[0].delta.content:
                    yield chunk.choices[0].delta.content
                    
        except Exception as e:
            logger.error(f"LLM generation error: {str(e)}")
            raise
    
    def build_section_prompt(
        self,
        section_name: str,
        metadata: Dict[str, Any],
        previous_content: str = ""
    ) -> tuple[str, str]:
        """
        Constructs system and user prompts for a specific section.
        """
        # System prompt remains consistent
        system_prompt = """You are a professional screenplay writer. 
Generate content that follows industry-standard formatting.
Focus solely on the requested section without adding commentary."""
        
        # User prompt varies by section
        if section_name == "title_page":
            user_prompt = f"""Write a properly formatted title page for a screenplay:

Title: {metadata.get('title', 'Untitled')}
Genre: {metadata.get('genre', 'Drama')}
Author: {metadata.get('author', 'Anonymous')}

Use standard screenplay title page formatting."""
        
        elif section_name == "act_1":
            user_prompt = f"""Write Act 1 of a {metadata.get('genre', 'drama')} screenplay.

Title: {metadata.get('title')}
Tone: {metadata.get('tone', 'balanced')}
Setting: {metadata.get('setting', 'contemporary')}

Previous sections:
{previous_content[:500] if previous_content else 'None'}

Write 20-25 pages of Act 1, establishing characters, world, and inciting incident."""
        
        elif section_name == "act_2a":
            user_prompt = f"""Continue with Act 2A (first half of Act 2).

Context from previous acts:
{previous_content[-1000:]}

Write 20-25 pages showing rising action and complications."""
        
        # Add more section types as needed
        
        else:
            user_prompt = f"Generate {section_name} section based on: {metadata}"
        
        return system_prompt, user_prompt
```

### 2.7 Section Orchestrator

```python
# app/services/orchestrator.py
from typing import List, Dict, Any, AsyncIterator
from app.models.generation import Section, Generation, SectionStatus, GenerationStatus
from app.models.schemas import SectionUpdate
from app.services.llm_service import LLMService
from app.utils.database import get_db
from sqlalchemy.orm import Session
import logging
import uuid

logger = logging.getLogger(__name__)

class SectionOrchestrator:
    """
    Manages the lifecycle of multi-section generation.
    """
    
    # Define section structure for each project type
    SECTION_TEMPLATES = {
        "screenplay": [
            {"name": "title_page", "order": 1, "max_tokens": 500},
            {"name": "act_1", "order": 2, "max_tokens": 4000},
            {"name": "act_2a", "order": 3, "max_tokens": 4000},
            {"name": "act_2b", "order": 4, "max_tokens": 4000},
            {"name": "act_3", "order": 5, "max_tokens": 4000},
        ],
        "novel_chapter": [
            {"name": "opening", "order": 1, "max_tokens": 2000},
            {"name": "development", "order": 2, "max_tokens": 3000},
            {"name": "climax", "order": 3, "max_tokens": 2000},
            {"name": "resolution", "order": 4, "max_tokens": 1500},
        ]
    }
    
    def __init__(self, llm_service: LLMService, db: Session):
        self.llm_service = llm_service
        self.db = db
    
    def initialize_generation(
        self,
        generation_id: str,
        project_type: str
    ) -> List[Section]:
        """
        Creates section records for a new generation.
        """
        template = self.SECTION_TEMPLATES.get(project_type, [])
        sections = []
        
        for section_def in template:
            section = Section(
                id=str(uuid.uuid4()),
                generation_id=generation_id,
                section_name=section_def["name"],
                section_order=section_def["order"],
                status=SectionStatus.PENDING
            )
            sections.append(section)
            self.db.add(section)
        
        self.db.commit()
        return sections
    
    async def generate_all_sections(
        self,
        generation_id: str
    ) -> AsyncIterator[SectionUpdate]:
        """
        Main orchestration loop - generates all sections sequentially.
        Yields updates after each chunk for streaming to client.
        """
        generation = self.db.query(Generation).filter_by(id=generation_id).first()
        if not generation:
            raise ValueError(f"Generation {generation_id} not found")
        
        # Update generation status
        generation.status = GenerationStatus.IN_PROGRESS
        self.db.commit()
        
        # Get all sections ordered
        sections = (
            self.db.query(Section)
            .filter_by(generation_id=generation_id)
            .order_by(Section.section_order)
            .all()
        )
        
        # Accumulate content for context
        full_content = ""
        
        for section in sections:
            try:
                # Signal section start
                yield SectionUpdate(
                    section_name=section.section_name,
                    status="generating",
                    is_complete=False
                )
                
                # Update section status
                section.status = SectionStatus.GENERATING
                self.db.commit()
                
                # Build prompts with context
                system_prompt, user_prompt = self.llm_service.build_section_prompt(
                    section_name=section.section_name,
                    metadata=generation.metadata,
                    previous_content=full_content
                )
                
                section.prompt_used = user_prompt
                
                # Stream generation
                section_content = ""
                token_count = 0
                
                async for chunk in self.llm_service.generate_section_stream(
                    system_prompt=system_prompt,
                    user_prompt=user_prompt,
                    max_tokens=self._get_section_max_tokens(
                        generation.project_type,
                        section.section_name
                    )
                ):
                    section_content += chunk
                    token_count += 1
                    
                    # Yield content update
                    yield SectionUpdate(
                        section_name=section.section_name,
                        status="generating",
                        content_chunk=chunk,
                        is_complete=False
                    )
                
                # Save completed section
                section.content = section_content
                section.status = SectionStatus.COMPLETED
                section.tokens_used = token_count
                full_content += f"\n\n{section_content}"
                self.db.commit()
                
                # Signal section completion
                yield SectionUpdate(
                    section_name=section.section_name,
                    status="completed",
                    is_complete=True,
                    tokens_used=token_count
                )
                
            except Exception as e:
                logger.error(f"Section {section.section_name} failed: {str(e)}")
                section.status = SectionStatus.FAILED
                section.error_message = str(e)
                section.retry_count += 1
                self.db.commit()
                
                # Yield error
                yield SectionUpdate(
                    section_name=section.section_name,
                    status="failed",
                    error=str(e),
                    is_complete=True
                )
                
                # Decide whether to continue or abort
                if section.retry_count >= 3:
                    generation.status = GenerationStatus.FAILED
                    self.db.commit()
                    break
        
        # Mark generation complete
        generation.status = GenerationStatus.COMPLETED
        self.db.commit()
    
    def _get_section_max_tokens(self, project_type: str, section_name: str) -> int:
        template = self.SECTION_TEMPLATES.get(project_type, [])
        for section in template:
            if section["name"] == section_name:
                return section.get("max_tokens", 2000)
        return 2000
```

### 2.8 WebSocket Stream Handler

```python
# app/api/websocket.py
from fastapi import WebSocket, WebSocketDisconnect, Depends
from app.services.orchestrator import SectionOrchestrator
from app.services.llm_service import LLMService
from app.utils.database import get_db
from app.config import get_settings
from sqlalchemy.orm import Session
import json
import logging
import asyncio

logger = logging.getLogger(__name__)
settings = get_settings()

class ConnectionManager:
    def __init__(self):
        self.active_connections: Dict[str, WebSocket] = {}
    
    async def connect(self, generation_id: str, websocket: WebSocket):
        await websocket.accept()
        self.active_connections[generation_id] = websocket
        logger.info(f"WebSocket connected for generation {generation_id}")
    
    def disconnect(self, generation_id: str):
        if generation_id in self.active_connections:
            del self.active_connections[generation_id]
            logger.info(f"WebSocket disconnected for generation {generation_id}")
    
    async def send_update(self, generation_id: str, message: dict):
        websocket = self.active_connections.get(generation_id)
        if websocket:
            try:
                await websocket.send_json(message)
            except Exception as e:
                logger.error(f"Failed to send update: {str(e)}")
                self.disconnect(generation_id)

manager = ConnectionManager()

async def stream_generation(
    websocket: WebSocket,
    generation_id: str,
    db: Session = Depends(get_db)
):
    await manager.connect(generation_id, websocket)
    
    try:
        # Initialize services
        llm_service = LLMService(
            api_key=settings.OPENAI_API_KEY,
            model=settings.OPENAI_MODEL
        )
        orchestrator = SectionOrchestrator(llm_service, db)
        
        # Start heartbeat task
        async def heartbeat():
            while True:
                try:
                    await websocket.send_json({"type": "heartbeat"})
                    await asyncio.sleep(settings.HEARTBEAT_INTERVAL)
                except:
                    break
        
        heartbeat_task = asyncio.create_task(heartbeat())
        
        # Stream generation
        async for update in orchestrator.generate_all_sections(generation_id):
            message = {
                "type": "section_update",
                "generation_id": generation_id,
                "section_name": update.section_name,
                "status": update.status,
                "content": update.content_chunk,
                "is_complete": update.is_complete,
                "tokens_used": update.tokens_used,
                "error": update.error
            }
            await manager.send_update(generation_id, message)
        
        # Send completion
        await manager.send_update(generation_id, {
            "type": "generation_complete",
            "generation_id": generation_id
        })
        
        heartbeat_task.cancel()
        
    except WebSocketDisconnect:
        logger.info(f"Client disconnected: {generation_id}")
    except Exception as e:
        logger.error(f"WebSocket error: {str(e)}")
        await manager.send_update(generation_id, {
            "type": "error",
            "message": str(e)
        })
    finally:
        manager.disconnect(generation_id)
```

### 2.9 REST API Endpoints

```python
# app/api/routes.py
from fastapi import APIRouter, Depends, HTTPException, BackgroundTasks
from sqlalchemy.orm import Session
from app.models.generation import Generation, GenerationStatus
from app.models.schemas import GenerationRequest, GenerationResponse
from app.utils.database import get_db
from app.services.orchestrator import SectionOrchestrator
from app.services.llm_service import LLMService
from app.config import get_settings
import uuid

router = APIRouter()
settings = get_settings()

@router.post("/generations", response_model=GenerationResponse)
async def create_generation(
    request: GenerationRequest,
    background_tasks: BackgroundTasks,
    db: Session = Depends(get_db)
):
    """
    Initiates a new generation and returns WebSocket connection info.
    """
    generation_id = str(uuid.uuid4())
    
    # Create generation record
    generation = Generation(
        id=generation_id,
        user_id=request.user_id,
        project_type=request.project_type,
        metadata=request.metadata,
        status=GenerationStatus.PENDING
    )
    db.add(generation)
    db.commit()
    
    # Initialize sections
    llm_service = LLMService(settings.OPENAI_API_KEY, settings.OPENAI_MODEL)
    orchestrator = SectionOrchestrator(llm_service, db)
    orchestrator.initialize_generation(generation_id, request.project_type)
    
    return GenerationResponse(
        generation_id=generation_id,
        status="pending",
        websocket_url=f"/ws/generate/{generation_id}"
    )

@router.get("/generations/{generation_id}")
async def get_generation(generation_id: str, db: Session = Depends(get_db)):
    """
    Retrieves the current state of a generation.
    """
    generation = db.query(Generation).filter_by(id=generation_id).first()
    if not generation:
        raise HTTPException(status_code=404, detail="Generation not found")
    
    sections = db.query(Section).filter_by(generation_id=generation_id).all()
    
    return {
        "id": generation.id,
        "status": generation.status,
        "sections": [
            {
                "name": s.section_name,
                "status": s.status,
                "content": s.content,
                "tokens_used": s.tokens_used
            }
            for s in sections
        ],
        "created_at": generation.created_at,
        "completed_at": generation.completed_at
    }

@router.post("/generations/{generation_id}/retry/{section_name}")
async def retry_section(
    generation_id: str,
    section_name: str,
    db: Session = Depends(get_db)
):
    """
    Retries a failed section.
    """
    section = (
        db.query(Section)
        .filter_by(generation_id=generation_id, section_name=section_name)
        .first()
    )
    
    if not section:
        raise HTTPException(status_code=404, detail="Section not found")
    
    section.status = SectionStatus.PENDING
    section.content = ""
    section.error_message = None
    db.commit()
    
    return {"message": "Section queued for retry"}
```

### 2.10 Main Application

```python
# app/main.py
from fastapi import FastAPI, WebSocket, Depends
from fastapi.middleware.cors import CORSMiddleware
from app.api.routes import router
from app.api.websocket import stream_generation
from app.utils.database import engine, Base
from app.config import get_settings

settings = get_settings()

# Create tables
Base.metadata.create_all(bind=engine)

app = FastAPI(title=settings.APP_NAME)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Configure appropriately for production
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Include routes
app.include_router(router, prefix="/api/v1")

# WebSocket endpoint
@app.websocket("/ws/generate/{generation_id}")
async def websocket_endpoint(websocket: WebSocket, generation_id: str):
    await stream_generation(websocket, generation_id)

@app.get("/health")
async def health_check():
    return {"status": "healthy"}
```

---

## 3. Flutter Frontend Integration

### 3.1 High-Level Architecture

```
UI Layer (Widgets)
      ↓
State Management (Riverpod/Bloc)
      ↓
Services Layer
      ↓
WebSocket Client
      ↓
Python Backend
```

### 3.2 Key Components

#### WebSocket Service

```dart
// lib/services/websocket_service.dart
import 'package:web_socket_channel/web_socket_channel.dart';
import 'dart:convert';
import 'dart:async';

class WebSocketService {
  WebSocketChannel? _channel;
  final StreamController<GenerationUpdate> _updateController = 
      StreamController<GenerationUpdate>.broadcast();
  
  Stream<GenerationUpdate> get updates => _updateController.stream;
  
  void connect(String generationId) {
    final wsUrl = 'ws://your-backend.com/ws/generate/$generationId';
    _channel = WebSocketChannel.connect(Uri.parse(wsUrl));
    
    _channel!.stream.listen(
      (message) {
        final data = jsonDecode(message);
        _updateController.add(GenerationUpdate.fromJson(data));
      },
      onError: (error) {
        _updateController.addError(error);
      },
      onDone: () {
        _updateController.close();
      },
    );
  }
  
  void disconnect() {
    _channel?.sink.close();
  }
}

class GenerationUpdate {
  final String type;
  final String? sectionName;
  final String? content;
  final bool isComplete;
  
  GenerationUpdate({
    required this.type,
    this.sectionName,
    this.content,
    this.isComplete = false,
  });
  
  factory GenerationUpdate.fromJson(Map<String, dynamic> json) {
    return GenerationUpdate(
      type: json['type'],
      sectionName: json['section_name'],
      content: json['content'],
      isComplete: json['is_complete'] ?? false,
    );
  }
}
```

#### State Management (Riverpod Example)

```dart
// lib/providers/generation_provider.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../services/websocket_service.dart';

class GenerationState {
  final String generationId;
  final Map<String, String> sectionContent;
  final String currentSection;
  final bool isComplete;
  
  GenerationState({
    required this.generationId,
    this.sectionContent = const {},
    this.currentSection = '',
    this.isComplete = false,
  });
  
  GenerationState copyWith({
    Map<String, String>? sectionContent,
    String? currentSection,
    bool? isComplete,
  }) {
    return GenerationState(
      generationId: generationId,
      sectionContent: sectionContent ?? this.sectionContent,
      currentSection: currentSection ?? this.currentSection,
      isComplete: isComplete ?? this.isComplete,
    );
  }
}

class GenerationNotifier extends StateNotifier<GenerationState> {
  final WebSocketService _wsService;
  
  GenerationNotifier(this._wsService, String generationId) 
      : super(GenerationState(generationId: generationId)) {
    _init();
  }
  
  void _init() {
    _wsService.connect(state.generationId);
    _wsService.updates.listen((update) {
      if (update.type == 'section_update') {
        _handleSectionUpdate(update);
      } else if (update.type == 'generation_complete') {
        state = state.copyWith(isComplete: true);
      }
    });
  }
  
  void _handleSectionUpdate(GenerationUpdate update) {
    if (update.content != null) {
      final updatedContent = Map<String, String>.from(state.sectionContent);
      updatedContent[update.sectionName!] = 
          (updatedContent[update.sectionName!] ?? '') + update.content!;
      
      state = state.copyWith(
        sectionContent: updatedContent,
        currentSection: update.sectionName,
      );
    }
  }
  
  @override
  void dispose() {
    _wsService.disconnect();
    super.dispose();
  }
}
```

#### UI Implementation

```dart
// lib/screens/generation_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

class GenerationScreen extends ConsumerWidget {
  final String generationId;
  
  const GenerationScreen({required this.generationId});
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final generationState = ref.watch(generationProvider(generationId));
    
    return Scaffold(
      appBar: AppBar(
        title: Text('Generating Content'),
        actions: [
          if (generationState.isComplete)
            IconButton(
              icon: Icon(Icons.download),
              onPressed: () => _exportContent(generationState),
            ),
        ],
      ),
      body: Column(
        children: [
          LinearProgressIndicator(
            value: generationState.isComplete ? 1.0 : null,
          ),
          Expanded(
            child: ListView.builder(
              itemCount: generationState.sectionContent.length,
              itemBuilder: (context, index) {
                final section = generationState.sectionContent.keys.elementAt(index);
                final content = generationState.sectionContent[section]!;
                
                return StreamingTextWidget(
                  sectionName: section,
                  content: content,
                  isActive: section == generationState.currentSection,
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}

class StreamingTextWidget extends StatefulWidget {
  final String sectionName;
  final String content;
  final bool isActive;
  
  const StreamingTextWidget({
    required this.sectionName,
    required this.content,
    required this.isActive,
  });
  
  @override
  State<StreamingTextWidget> createState() => _StreamingTextWidgetState();
}

class _StreamingTextWidgetState extends State<StreamingTextWidget> 
    with SingleTickerProviderStateMixin {
  final ScrollController _scrollController = ScrollController();
  late AnimationController _cursorController;
  
  @override
  void initState() {
    super.initState();
    _cursorController = AnimationController(
      vsync: this,
      duration: Duration(milliseconds: 500),
    )..repeat(reverse: true);
  }
  
  @override
  void didUpdateWidget(StreamingTextWidget oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (widget.content != oldWidget.content && widget.isActive) {
      // Auto-scroll to bottom when new content arrives
      WidgetsBinding.instance.addPostFrameCallback((_) {
        if (_scrollController.hasClients) {
          _scrollController.animateTo(
            _scrollController.position.maxScrollExtent,
            duration: Duration(milliseconds: 300),
            curve: Curves.easeOut,
          );
        }
      });
    }
  }
  
  @override
  void dispose() {
    _cursorController.dispose();
    _scrollController.dispose();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return Card(
      margin: EdgeInsets.all(8),
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              children: [
                Text(
                  _formatSectionName(widget.sectionName),
                  style: Theme.of(context).textTheme.titleMedium?.copyWith(
                    fontWeight: FontWeight.bold,
                  ),
                ),
                Spacer(),
                if (widget.isActive)
                  FadeTransition(
                    opacity: _cursorController,
                    child: Icon(Icons.edit, size: 16, color: Colors.blue),
                  ),
              ],
            ),
            SizedBox(height: 12),
            ConstrainedBox(
              constraints: BoxConstraints(maxHeight: 400),
              child: SingleChildScrollView(
                controller: _scrollController,
                child: SelectableText(
                  widget.content,
                  style: TextStyle(
                    fontFamily: 'Courier',
                    fontSize: 14,
                    height: 1.5,
                  ),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
  
  String _formatSectionName(String name) {
    return name
        .split('_')
        .map((word) => word[0].toUpperCase() + word.substring(1))
        .join(' ');
  }
}
```

#### HTTP Service for API Calls

```dart
// lib/services/api_service.dart
import 'package:http/http.dart' as http;
import 'dart:convert';

class ApiService {
  final String baseUrl;
  
  ApiService({required this.baseUrl});
  
  Future<String> createGeneration({
    required String userId,
    required String projectType,
    required Map<String, dynamic> metadata,
  }) async {
    final response = await http.post(
      Uri.parse('$baseUrl/api/v1/generations'),
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode({
        'user_id': userId,
        'project_type': projectType,
        'metadata': metadata,
      }),
    );
    
    if (response.statusCode == 200) {
      final data = jsonDecode(response.body);
      return data['generation_id'];
    } else {
      throw Exception('Failed to create generation');
    }
  }
  
  Future<Map<String, dynamic>> getGeneration(String generationId) async {
    final response = await http.get(
      Uri.parse('$baseUrl/api/v1/generations/$generationId'),
    );
    
    if (response.statusCode == 200) {
      return jsonDecode(response.body);
    } else {
      throw Exception('Failed to fetch generation');
    }
  }
  
  Future<void> retrySection({
    required String generationId,
    required String sectionName,
  }) async {
    final response = await http.post(
      Uri.parse('$baseUrl/api/v1/generations/$generationId/retry/$sectionName'),
    );
    
    if (response.statusCode != 200) {
      throw Exception('Failed to retry section');
    }
  }
}
```

---

## 4. Production Considerations

### 4.1 Error Recovery & Retry Logic

```python
# app/services/recovery_service.py
from typing import Optional
from app.models.generation import Section, SectionStatus
from app.services.llm_service import LLMService
from sqlalchemy.orm import Session
import logging
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type

logger = logging.getLogger(__name__)

class RecoveryService:
    """
    Handles recovery from failures during generation.
    """
    
    def __init__(self, db: Session, llm_service: LLMService):
        self.db = db
        self.llm_service = llm_service
    
    async def resume_generation(self, generation_id: str) -> bool:
        """
        Resumes a paused or failed generation from the last successful section.
        """
        sections = (
            self.db.query(Section)
            .filter_by(generation_id=generation_id)
            .order_by(Section.section_order)
            .all()
        )
        
        # Find first incomplete section
        for section in sections:
            if section.status in [SectionStatus.PENDING, SectionStatus.FAILED]:
                logger.info(f"Resuming from section: {section.section_name}")
                return True
        
        return False
    
    @retry(
        stop=stop_after_attempt(3),
        wait=wait_exponential(multiplier=2, min=4, max=30),
        retry=retry_if_exception_type((ConnectionError, TimeoutError))
    )
    async def retry_section_with_backoff(
        self,
        section: Section,
        system_prompt: str,
        user_prompt: str
    ) -> str:
        """
        Retries a section generation with exponential backoff.
        """
        try:
            content = ""
            async for chunk in self.llm_service.generate_section_stream(
                system_prompt=system_prompt,
                user_prompt=user_prompt
            ):
                content += chunk
            
            return content
        except Exception as e:
            section.retry_count += 1
            section.error_message = str(e)
            self.db.commit()
            raise
```

### 4.2 Redis Caching & Session Management

```python
# app/utils/redis_client.py
import redis
import json
from typing import Optional, Any
from app.config import get_settings

settings = get_settings()

class RedisClient:
    def __init__(self):
        self.client = redis.from_url(settings.REDIS_URL, decode_responses=True)
    
    def cache_section(self, generation_id: str, section_name: str, content: str):
        """
        Caches section content for quick recovery.
        """
        key = f"section:{generation_id}:{section_name}"
        self.client.setex(key, 3600, content)  # 1 hour expiry
    
    def get_cached_section(self, generation_id: str, section_name: str) -> Optional[str]:
        """
        Retrieves cached section content.
        """
        key = f"section:{generation_id}:{section_name}"
        return self.client.get(key)
    
    def store_generation_state(self, generation_id: str, state: dict):
        """
        Stores generation state for recovery.
        """
        key = f"state:{generation_id}"
        self.client.setex(key, 7200, json.dumps(state))
    
    def get_generation_state(self, generation_id: str) -> Optional[dict]:
        """
        Retrieves generation state.
        """
        key = f"state:{generation_id}"
        data = self.client.get(key)
        return json.loads(data) if data else None
    
    def check_rate_limit(self, user_id: str) -> bool:
        """
        Checks if user has exceeded concurrent generation limit.
        """
        key = f"ratelimit:{user_id}"
        current = self.client.get(key)
        if current and int(current) >= settings.RATE_LIMIT_PER_USER:
            return False
        return True
    
    def increment_rate_limit(self, user_id: str):
        """
        Increments user's active generation count.
        """
        key = f"ratelimit:{user_id}"
        self.client.incr(key)
        self.client.expire(key, 3600)
    
    def decrement_rate_limit(self, user_id: str):
        """
        Decrements user's active generation count.
        """
        key = f"ratelimit:{user_id}"
        self.client.decr(key)
```

### 4.3 Background Task Processing with Celery

```python
# app/tasks/celery_tasks.py
from celery import Celery
from app.config import get_settings
from app.utils.database import SessionLocal
from app.services.llm_service import LLMService
from app.services.orchestrator import SectionOrchestrator
from app.models.generation import Generation, GenerationStatus
import logging

settings = get_settings()
logger = logging.getLogger(__name__)

celery_app = Celery(
    'tasks',
    broker=settings.REDIS_URL,
    backend=settings.REDIS_URL
)

celery_app.conf.update(
    task_serializer='json',
    accept_content=['json'],
    result_serializer='json',
    timezone='UTC',
    enable_utc=True,
)

@celery_app.task(bind=True, max_retries=3)
def process_generation_async(self, generation_id: str):
    """
    Background task for processing generations without blocking HTTP requests.
    Used for non-realtime scenarios or as fallback.
    """
    db = SessionLocal()
    try:
        generation = db.query(Generation).filter_by(id=generation_id).first()
        if not generation:
            logger.error(f"Generation {generation_id} not found")
            return
        
        generation.status = GenerationStatus.IN_PROGRESS
        db.commit()
        
        llm_service = LLMService(settings.OPENAI_API_KEY, settings.OPENAI_MODEL)
        orchestrator = SectionOrchestrator(llm_service, db)
        
        # Process synchronously in background
        import asyncio
        loop = asyncio.new_event_loop()
        asyncio.set_event_loop(loop)
        
        async def run_generation():
            async for update in orchestrator.generate_all_sections(generation_id):
                logger.info(f"Section update: {update.section_name} - {update.status}")
        
        loop.run_until_complete(run_generation())
        
        generation.status = GenerationStatus.COMPLETED
        db.commit()
        
    except Exception as e:
        logger.error(f"Background generation failed: {str(e)}")
        generation.status = GenerationStatus.FAILED
        generation.error_message = str(e)
        db.commit()
        
        # Retry with exponential backoff
        self.retry(countdown=60 * (2 ** self.request.retries))
    
    finally:
        db.close()

@celery_app.task
def cleanup_old_generations():
    """
    Periodic task to clean up old completed/failed generations.
    """
    db = SessionLocal()
    try:
        from datetime import datetime, timedelta
        cutoff = datetime.utcnow() - timedelta(days=30)
        
        old_generations = (
            db.query(Generation)
            .filter(Generation.created_at < cutoff)
            .filter(Generation.status.in_([
                GenerationStatus.COMPLETED,
                GenerationStatus.FAILED
            ]))
            .all()
        )
        
        for gen in old_generations:
            db.delete(gen)
        
        db.commit()
        logger.info(f"Cleaned up {len(old_generations)} old generations")
        
    finally:
        db.close()

# Celery beat schedule for periodic tasks
celery_app.conf.beat_schedule = {
    'cleanup-every-day': {
        'task': 'app.tasks.celery_tasks.cleanup_old_generations',
        'schedule': 86400.0,  # 24 hours
    },
}
```

### 4.4 Database Connection Pooling

```python
# app/utils/database.py
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, Session
from sqlalchemy.pool import QueuePool
from app.config import get_settings
from contextlib import contextmanager

settings = get_settings()

# Engine with connection pooling
engine = create_engine(
    settings.DATABASE_URL,
    poolclass=QueuePool,
    pool_size=10,           # Number of persistent connections
    max_overflow=20,        # Additional connections when pool is full
    pool_timeout=30,        # Timeout waiting for connection
    pool_recycle=3600,      # Recycle connections after 1 hour
    pool_pre_ping=True,     # Verify connections before using
    echo=settings.DEBUG
)

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

def get_db() -> Session:
    """
    Dependency for FastAPI routes to get database session.
    """
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@contextmanager
def get_db_context():
    """
    Context manager for use outside FastAPI (e.g., Celery tasks).
    """
    db = SessionLocal()
    try:
        yield db
        db.commit()
    except Exception:
        db.rollback()
        raise
    finally:
        db.close()
```

### 4.5 Monitoring & Logging

```python
# app/utils/monitoring.py
import logging
import time
from functools import wraps
from typing import Callable
import structlog

# Configure structured logging
structlog.configure(
    processors=[
        structlog.stdlib.filter_by_level,
        structlog.stdlib.add_logger_name,
        structlog.stdlib.add_log_level,
        structlog.stdlib.PositionalArgumentsFormatter(),
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.StackInfoRenderer(),
        structlog.processors.format_exc_info,
        structlog.processors.UnicodeDecoder(),
        structlog.processors.JSONRenderer()
    ],
    context_class=dict,
    logger_factory=structlog.stdlib.LoggerFactory(),
    cache_logger_on_first_use=True,
)

logger = structlog.get_logger()

def track_performance(func: Callable):
    """
    Decorator to track function execution time and log performance metrics.
    """
    @wraps(func)
    async def async_wrapper(*args, **kwargs):
        start_time = time.time()
        try:
            result = await func(*args, **kwargs)
            duration = time.time() - start_time
            logger.info(
                "function_executed",
                function=func.__name__,
                duration_seconds=duration,
                status="success"
            )
            return result
        except Exception as e:
            duration = time.time() - start_time
            logger.error(
                "function_failed",
                function=func.__name__,
                duration_seconds=duration,
                error=str(e),
                status="error"
            )
            raise
    
    @wraps(func)
    def sync_wrapper(*args, **kwargs):
        start_time = time.time()
        try:
            result = func(*args, **kwargs)
            duration = time.time() - start_time
            logger.info(
                "function_executed",
                function=func.__name__,
                duration_seconds=duration,
                status="success"
            )
            return result
        except Exception as e:
            duration = time.time() - start_time
            logger.error(
                "function_failed",
                function=func.__name__,
                duration_seconds=duration,
                error=str(e),
                status="error"
            )
            raise
    
    import asyncio
    if asyncio.iscoroutinefunction(func):
        return async_wrapper
    else:
        return sync_wrapper

class GenerationMetrics:
    """
    Tracks metrics for generation operations.
    """
    
    @staticmethod
    def log_section_metrics(
        generation_id: str,
        section_name: str,
        tokens_used: int,
        duration_seconds: float,
        status: str
    ):
        logger.info(
            "section_completed",
            generation_id=generation_id,
            section_name=section_name,
            tokens_used=tokens_used,
            duration_seconds=duration_seconds,
            tokens_per_second=tokens_used / duration_seconds if duration_seconds > 0 else 0,
            status=status
        )
    
    @staticmethod
    def log_generation_metrics(
        generation_id: str,
        total_sections: int,
        completed_sections: int,
        total_tokens: int,
        total_duration: float,
        status: str
    ):
        logger.info(
            "generation_completed",
            generation_id=generation_id,
            total_sections=total_sections,
            completed_sections=completed_sections,
            total_tokens=total_tokens,
            total_duration_seconds=total_duration,
            average_tokens_per_section=total_tokens / total_sections if total_sections > 0 else 0,
            status=status
        )
```

### 4.6 Security Considerations

```python
# app/middleware/security.py
from fastapi import Request, HTTPException
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from typing import Optional
import jwt
from app.config import get_settings

settings = get_settings()
security = HTTPBearer()

class AuthMiddleware:
    """
    JWT-based authentication middleware.
    """
    
    @staticmethod
    def verify_token(token: str) -> Optional[dict]:
        try:
            payload = jwt.decode(
                token,
                settings.JWT_SECRET,
                algorithms=["HS256"]
            )
            return payload
        except jwt.ExpiredSignatureError:
            raise HTTPException(status_code=401, detail="Token expired")
        except jwt.InvalidTokenError:
            raise HTTPException(status_code=401, detail="Invalid token")
    
    @staticmethod
    async def authenticate(credentials: HTTPAuthorizationCredentials):
        token = credentials.credentials
        payload = AuthMiddleware.verify_token(token)
        return payload

# Add to routes that require authentication:
# @router.post("/generations", dependencies=[Depends(security)])

class RateLimiter:
    """
    Rate limiting to prevent abuse.
    """
    
    def __init__(self, redis_client):
        self.redis = redis_client
    
    async def check_limit(self, user_id: str, endpoint: str) -> bool:
        key = f"ratelimit:{endpoint}:{user_id}"
        current = await self.redis.incr(key)
        
        if current == 1:
            await self.redis.expire(key, 60)  # 1 minute window
        
        if current > 10:  # 10 requests per minute
            raise HTTPException(
                status_code=429,
                detail="Rate limit exceeded"
            )
        
        return True
```

---

## 5. Deployment & Scaling

### 5.1 Docker Configuration

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Run migrations and start server
CMD ["sh", "-c", "alembic upgrade head && uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: streaming_db
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
  
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
  
  backend:
    build: .
    environment:
      DATABASE_URL: postgresql://admin:${DB_PASSWORD}@postgres:5432/streaming_db
      REDIS_URL: redis://redis:6379/0
      OPENAI_API_KEY: ${OPENAI_API_KEY}
    ports:
      - "8000:8000"
    depends_on:
      - postgres
      - redis
    volumes:
      - ./app:/app/app
  
  celery_worker:
    build: .
    command: celery -A app.tasks.celery_tasks worker --loglevel=info
    environment:
      DATABASE_URL: postgresql://admin:${DB_PASSWORD}@postgres:5432/streaming_db
      REDIS_URL: redis://redis:6379/0
      OPENAI_API_KEY: ${OPENAI_API_KEY}
    depends_on:
      - postgres
      - redis
  
  celery_beat:
    build: .
    command: celery -A app.tasks.celery_tasks beat --loglevel=info
    environment:
      DATABASE_URL: postgresql://admin:${DB_PASSWORD}@postgres:5432/streaming_db
      REDIS_URL: redis://redis:6379/0
    depends_on:
      - postgres
      - redis

volumes:
  postgres_data:
  redis_data:
```

### 5.2 Production Environment Variables

```bash
# .env.production
APP_NAME=Section-Based Streaming API
DEBUG=false

# Database
DATABASE_URL=postgresql://user:password@db-host:5432/production_db
DB_POOL_SIZE=20
DB_MAX_OVERFLOW=40

# Redis
REDIS_URL=redis://redis-host:6379/0

# LLM
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-4-turbo-preview
MAX_TOKENS_PER_SECTION=4000

# Security
JWT_SECRET=your-secret-key-here
ALLOWED_ORIGINS=https://yourdomain.com,https://app.yourdomain.com

# Monitoring
SENTRY_DSN=https://...
LOG_LEVEL=INFO

# Rate Limiting
RATE_LIMIT_PER_USER=5
MAX_RETRIES=3
```

### 5.3 Scaling Strategies

#### Horizontal Scaling with Load Balancer

```nginx
# nginx.conf
upstream backend {
    least_conn;
    server backend1:8000 weight=1;
    server backend2:8000 weight=1;
    server backend3:8000 weight=1;
}

server {
    listen 80;
    server_name api.yourdomain.com;
    
    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        # WebSocket support
        proxy_read_timeout 3600s;
        proxy_send_timeout 3600s;
    }
}
```

#### Kubernetes Deployment

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: streaming-backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: streaming-backend
  template:
    metadata:
      labels:
        app: streaming-backend
    spec:
      containers:
      - name: backend
        image: your-registry/streaming-backend:latest
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: database-url
        - name: OPENAI_API_KEY
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: openai-key
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: streaming-backend-service
spec:
  selector:
    app: streaming-backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
  type: LoadBalancer
```

### 5.4 Database Optimization

```python
# app/utils/database_optimization.py
from sqlalchemy import Index, text
from app.models.generation import Generation, Section

# Add indexes for common queries
def create_indexes(engine):
    """
    Creates database indexes for performance optimization.
    """
    with engine.connect() as conn:
        # Index on generation user_id and status for dashboard queries
        conn.execute(text(
            "CREATE INDEX IF NOT EXISTS idx_generations_user_status "
            "ON generations(user_id, status)"
        ))
        
        # Index on sections for generation retrieval
        conn.execute(text(
            "CREATE INDEX IF NOT EXISTS idx_sections_generation_order "
            "ON sections(generation_id, section_order)"
        ))
        
        # Index on created_at for cleanup tasks
        conn.execute(text(
            "CREATE INDEX IF NOT EXISTS idx_generations_created "
            "ON generations(created_at)"
        ))
        
        conn.commit()

# Optimize queries with eager loading
from sqlalchemy.orm import joinedload

def get_generation_with_sections(db: Session, generation_id: str):
    """
    Efficiently loads generation with all sections in one query.
    """
    return (
        db.query(Generation)
        .options(joinedload(Generation.sections))
        .filter_by(id=generation_id)
        .first()
    )
```

### 5.5 Monitoring & Alerting

```python
# app/utils/alerting.py
import sentry_sdk
from sentry_sdk.integrations.fastapi import FastApiIntegration
from sentry_sdk.integrations.sqlalchemy import SqlalchemyIntegration
from app.config import get_settings

settings = get_settings()

def initialize_monitoring():
    """
    Initializes Sentry for error tracking and performance monitoring.
    """
    sentry_sdk.init(
        dsn=settings.SENTRY_DSN,
        integrations=[
            FastApiIntegration(),
            SqlalchemyIntegration(),
        ],
        traces_sample_rate=0.1,  # 10% of transactions for performance monitoring
        profiles_sample_rate=0.1,
        environment=settings.ENVIRONMENT,
    )

# Custom metrics for business KPIs
class Metrics:
    @staticmethod
    def track_generation_started(generation_id: str, user_id: str, project_type: str):
        logger.info(
            "generation_started",
            generation_id=generation_id,
            user_id=user_id,
            project_type=project_type
        )
    
    @staticmethod
    def track_generation_completed(
        generation_id: str,
        duration_seconds: float,
        total_tokens: int,
        sections_count: int
    ):
        logger.info(
            "generation_completed",
            generation_id=generation_id,
            duration_seconds=duration_seconds,
            total_tokens=total_tokens,
            sections_count=sections_count,
            cost_estimate=total_tokens * 0.00003  # Example pricing
        )
```

---

## 6. Testing Strategy

### 6.1 Unit Tests

```python
# tests/test_orchestrator.py
import pytest
from unittest.mock import Mock, AsyncMock, patch
from app.services.orchestrator import SectionOrchestrator
from app.models.generation import Generation, Section, SectionStatus

@pytest.mark.asyncio
async def test_initialize_generation():
    mock_db = Mock()
    mock_llm = Mock()
    
    orchestrator = SectionOrchestrator(mock_llm, mock_db)
    sections = orchestrator.initialize_generation("gen_123", "screenplay")
    
    assert len(sections) == 5
    assert sections[0].section_name == "title_page"
    assert sections[4].section_name == "act_3"

@pytest.mark.asyncio
async def test_section_streaming():
    mock_llm = AsyncMock()
    mock_llm.generate_section_stream = AsyncMock()
    mock_llm.generate_section_stream.return_value = async_generator(["Hello", " world"])
    
    # Test streaming accumulation
    content = ""
    async for chunk in mock_llm.generate_section_stream("sys", "user"):
        content += chunk
    
    assert content == "Hello world"

async def async_generator(items):
    for item in items:
        yield item
```

### 6.2 Integration Tests

```python
# tests/test_api.py
import pytest
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_create_generation():
    response = client.post(
        "/api/v1/generations",
        json={
            "user_id": "test_user",
            "project_type": "screenplay",
            "metadata": {"title": "Test Script"}
        }
    )
    
    assert response.status_code == 200
    data = response.json()
    assert "generation_id" in data
    assert "websocket_url" in data

def test_get_generation():
    # First create
    create_response = client.post(
        "/api/v1/generations",
        json={"user_id": "test", "project_type": "screenplay", "metadata": {}}
    )
    gen_id = create_response.json()["generation_id"]
    
    # Then retrieve
    response = client.get(f"/api/v1/generations/{gen_id}")
    assert response.status_code == 200
    assert response.json()["id"] == gen_id
```

### 6.3 Load Testing

```python
# tests/load_test.py
```python
# tests/load_test.py
import asyncio
import aiohttp
import time
from typing import List
import statistics

async def simulate_generation(session: aiohttp.ClientSession, user_id: str):
    """
    Simulates a single user generating content.
    """
    start_time = time.time()
    
    try:
        # Create generation
        async with session.post(
            'http://localhost:8000/api/v1/generations',
            json={
                'user_id': user_id,
                'project_type': 'screenplay',
                'metadata': {'title': 'Load Test Script'}
            }
        ) as response:
            if response.status != 200:
                return {'success': False, 'duration': 0}
            
            data = await response.json()
            generation_id = data['generation_id']
        
        # Connect to WebSocket
        ws_url = f'ws://localhost:8000/ws/generate/{generation_id}'
        async with session.ws_connect(ws_url) as ws:
            sections_received = 0
            
            async for msg in ws:
                if msg.type == aiohttp.WSMsgType.TEXT:
                    data = msg.json()
                    if data['type'] == 'section_complete':
                        sections_received += 1
                    elif data['type'] == 'generation_complete':
                        break
                elif msg.type == aiohttp.WSMsgType.ERROR:
                    return {'success': False, 'duration': time.time() - start_time}
        
        duration = time.time() - start_time
        return {
            'success': True,
            'duration': duration,
            'sections': sections_received
        }
    
    except Exception as e:
        print(f"Error: {str(e)}")
        return {'success': False, 'duration': time.time() - start_time}

async def load_test(concurrent_users: int, total_requests: int):
    """
    Runs load test with specified concurrency.
    """
    print(f"\nStarting load test: {concurrent_users} concurrent users, {total_requests} total requests")
    
    async with aiohttp.ClientSession() as session:
        results = []
        
        # Create batches of concurrent requests
        for batch_start in range(0, total_requests, concurrent_users):
            batch_size = min(concurrent_users, total_requests - batch_start)
            
            tasks = [
                simulate_generation(session, f'user_{i}')
                for i in range(batch_start, batch_start + batch_size)
            ]
            
            batch_results = await asyncio.gather(*tasks)
            results.extend(batch_results)
            
            # Progress update
            print(f"Completed {len(results)}/{total_requests} requests")
    
    # Calculate statistics
    successful = [r for r in results if r['success']]
    failed = [r for r in results if not r['success']]
    
    if successful:
        durations = [r['duration'] for r in successful]
        
        print("\n=== Load Test Results ===")
        print(f"Total Requests: {total_requests}")
        print(f"Successful: {len(successful)}")
        print(f"Failed: {len(failed)}")
        print(f"Success Rate: {len(successful)/total_requests*100:.2f}%")
        print(f"\nDuration Statistics:")
        print(f"  Min: {min(durations):.2f}s")
        print(f"  Max: {max(durations):.2f}s")
        print(f"  Mean: {statistics.mean(durations):.2f}s")
        print(f"  Median: {statistics.median(durations):.2f}s")
        print(f"  Std Dev: {statistics.stdev(durations):.2f}s")

if __name__ == '__main__':
    # Run load test
    asyncio.run(load_test(concurrent_users=10, total_requests=50))
```

---

## 7. Advanced Features

### 7.1 Pause and Resume Functionality

```python
# app/api/routes.py (additional endpoints)

@router.post("/generations/{generation_id}/pause")
async def pause_generation(
    generation_id: str,
    db: Session = Depends(get_db)
):
    """
    Pauses an ongoing generation.
    """
    generation = db.query(Generation).filter_by(id=generation_id).first()
    if not generation:
        raise HTTPException(status_code=404, detail="Generation not found")
    
    if generation.status != GenerationStatus.IN_PROGRESS:
        raise HTTPException(status_code=400, detail="Generation is not in progress")
    
    generation.status = GenerationStatus.PAUSED
    db.commit()
    
    # Signal to WebSocket to stop processing
    from app.api.websocket import manager
    await manager.send_update(generation_id, {
        "type": "paused",
        "generation_id": generation_id
    })
    
    return {"message": "Generation paused successfully"}

@router.post("/generations/{generation_id}/resume")
async def resume_generation(
    generation_id: str,
    background_tasks: BackgroundTasks,
    db: Session = Depends(get_db)
):
    """
    Resumes a paused generation.
    """
    generation = db.query(Generation).filter_by(id=generation_id).first()
    if not generation:
        raise HTTPException(status_code=404, detail="Generation not found")
    
    if generation.status != GenerationStatus.PAUSED:
        raise HTTPException(status_code=400, detail="Generation is not paused")
    
    # Use Celery to resume in background
    from app.tasks.celery_tasks import resume_generation_task
    background_tasks.add_task(resume_generation_task, generation_id)
    
    return {
        "message": "Generation resuming",
        "websocket_url": f"/ws/generate/{generation_id}"
    }
```

### 7.2 Section Regeneration

```python
# app/api/routes.py (additional endpoint)

@router.post("/generations/{generation_id}/regenerate/{section_name}")
async def regenerate_section(
    generation_id: str,
    section_name: str,
    db: Session = Depends(get_db)
):
    """
    Regenerates a specific section with different parameters.
    Useful when user is unhappy with a particular section.
    """
    section = (
        db.query(Section)
        .filter_by(generation_id=generation_id, section_name=section_name)
        .first()
    )
    
    if not section:
        raise HTTPException(status_code=404, detail="Section not found")
    
    # Reset section state
    section.status = SectionStatus.PENDING
    section.content = ""
    section.error_message = None
    section.retry_count = 0
    db.commit()
    
    return {
        "message": f"Section {section_name} queued for regeneration",
        "websocket_url": f"/ws/generate/{generation_id}"
    }
```

### 7.3 Real-time Progress Tracking

```python
# app/services/progress_tracker.py
from typing import Dict, Optional
from app.utils.redis_client import RedisClient
import json

class ProgressTracker:
    """
    Tracks generation progress in Redis for real-time updates.
    """
    
    def __init__(self, redis_client: RedisClient):
        self.redis = redis_client
    
    def update_progress(
        self,
        generation_id: str,
        current_section: int,
        total_sections: int,
        current_section_name: str,
        tokens_generated: int
    ):
        """
        Updates progress information in Redis.
        """
        progress_data = {
            'current_section': current_section,
            'total_sections': total_sections,
            'current_section_name': current_section_name,
            'tokens_generated': tokens_generated,
            'percentage': (current_section / total_sections) * 100
        }
        
        key = f"progress:{generation_id}"
        self.redis.client.setex(key, 3600, json.dumps(progress_data))
    
    def get_progress(self, generation_id: str) -> Optional[Dict]:
        """
        Retrieves current progress.
        """
        key = f"progress:{generation_id}"
        data = self.redis.client.get(key)
        return json.loads(data) if data else None

# Add to websocket stream
async def send_progress_update(generation_id: str, progress_tracker: ProgressTracker):
    progress = progress_tracker.get_progress(generation_id)
    if progress:
        await manager.send_update(generation_id, {
            "type": "progress",
            "data": progress
        })
```

### 7.4 Content Validation and Quality Checks

```python
# app/services/validation_service.py
from typing import Dict, List
import re

class ContentValidator:
    """
    Validates generated content for quality and format compliance.
    """
    
    @staticmethod
    def validate_screenplay_section(section_name: str, content: str) -> Dict[str, any]:
        """
        Validates screenplay formatting and content quality.
        """
        errors = []
        warnings = []
        
        # Check minimum length
        if len(content.strip()) < 100:
            errors.append(f"Section {section_name} is too short")
        
        # Check for proper screenplay formatting
        if section_name != "title_page":
            if not re.search(r'(INT\.|EXT\.)', content, re.IGNORECASE):
                warnings.append("No scene headings found")
            
            if not re.search(r'^[A-Z\s]+$', content, re.MULTILINE):
                warnings.append("Missing character names in proper format")
        
        # Check for placeholder text
        placeholders = ['[INSERT]', 'TODO', 'XXX', '...']
        for placeholder in placeholders:
            if placeholder in content:
                warnings.append(f"Contains placeholder text: {placeholder}")
        
        # Check for repetitive content
        lines = content.split('\n')
        unique_lines = set(lines)
        if len(unique_lines) < len(lines) * 0.8:
            warnings.append("High level of repetitive content detected")
        
        return {
            'valid': len(errors) == 0,
            'errors': errors,
            'warnings': warnings,
            'word_count': len(content.split()),
            'character_count': len(content)
        }
    
    @staticmethod
    def auto_fix_common_issues(content: str) -> str:
        """
        Automatically fixes common formatting issues.
        """
        # Remove excessive whitespace
        content = re.sub(r'\n{3,}', '\n\n', content)
        
        # Fix common typography
        content = content.replace(' .', '.')
        content = content.replace(' ,', ',')
        content = re.sub(r'\s+', ' ', content)
        
        # Ensure proper spacing after periods
        content = re.sub(r'\.([A-Z])', r'. \1', content)
        
        return content
```

### 7.5 Export Functionality

```python
# app/services/export_service.py
from typing import List
from app.models.generation import Generation, Section
from sqlalchemy.orm import Session
import io
from datetime import datetime

class ExportService:
    """
    Handles exporting generated content in various formats.
    """
    
    @staticmethod
    def export_as_text(generation_id: str, db: Session) -> str:
        """
        Exports generation as plain text.
        """
        generation = db.query(Generation).filter_by(id=generation_id).first()
        sections = (
            db.query(Section)
            .filter_by(generation_id=generation_id)
            .order_by(Section.section_order)
            .all()
        )
        
        output = f"Generated: {generation.created_at}\n"
        output += f"Type: {generation.project_type}\n"
        output += "=" * 80 + "\n\n"
        
        for section in sections:
            output += f"\n{'=' * 80}\n"
            output += f"{section.section_name.upper()}\n"
            output += f"{'=' * 80}\n\n"
            output += section.content + "\n"
        
        return output
    
    @staticmethod
    def export_as_fountain(generation_id: str, db: Session) -> str:
        """
        Exports screenplay in Fountain format.
        """
        sections = (
            db.query(Section)
            .filter_by(generation_id=generation_id)
            .order_by(Section.section_order)
            .all()
        )
        
        fountain_content = ""
        
        for section in sections:
            if section.section_name == "title_page":
                # Fountain title page format
                fountain_content += f"Title: {section.content}\n"
                fountain_content += f"Author: Generated\n"
                fountain_content += f"Draft date: {datetime.now().strftime('%m/%d/%Y')}\n\n"
            else:
                fountain_content += section.content + "\n\n"
        
        return fountain_content
    
    @staticmethod
    def export_as_pdf(generation_id: str, db: Session) -> bytes:
        """
        Exports as PDF using reportlab or similar library.
        Note: Requires additional dependencies like reportlab.
        """
        # This is a simplified example
        # In production, use proper PDF generation library
        
        from reportlab.lib.pagesizes import letter
        from reportlab.pdfgen import canvas
        
        buffer = io.BytesIO()
        c = canvas.Canvas(buffer, pagesize=letter)
        
        text = ExportService.export_as_text(generation_id, db)
        
        # Simple text rendering (production version needs proper formatting)
        y = 750
        for line in text.split('\n'):
            c.drawString(50, y, line[:80])  # Truncate long lines
            y -= 15
            if y < 50:
                c.showPage()
                y = 750
        
        c.save()
        buffer.seek(0)
        return buffer.read()

# Add export endpoint
@router.get("/generations/{generation_id}/export/{format}")
async def export_generation(
    generation_id: str,
    format: str,
    db: Session = Depends(get_db)
):
    """
    Exports generation in specified format.
    Formats: text, fountain, pdf
    """
    from app.services.export_service import ExportService
    from fastapi.responses import Response, PlainTextResponse
    
    if format == "text":
        content = ExportService.export_as_text(generation_id, db)
        return PlainTextResponse(content, media_type="text/plain")
    
    elif format == "fountain":
        content = ExportService.export_as_fountain(generation_id, db)
        return PlainTextResponse(content, media_type="text/plain")
    
    elif format == "pdf":
        content = ExportService.export_as_pdf(generation_id, db)
        return Response(content, media_type="application/pdf")
    
    else:
        raise HTTPException(status_code=400, detail="Invalid export format")
```

---

## 8. Flutter Frontend - Advanced Features

### 8.1 Offline Support with Local Caching

```dart
// lib/services/cache_service.dart
import 'package:hive_flutter/hive_flutter.dart';
import 'dart:convert';

class CacheService {
  static const String GENERATIONS_BOX = 'generations';
  
  Future<void> initialize() async {
    await Hive.initFlutter();
    await Hive.openBox(GENERATIONS_BOX);
  }
  
  Future<void> cacheGeneration(String generationId, Map<String, dynamic> data) async {
    final box = Hive.box(GENERATIONS_BOX);
    await box.put(generationId, jsonEncode(data));
  }
  
  Map<String, dynamic>? getCachedGeneration(String generationId) {
    final box = Hive.box(GENERATIONS_BOX);
    final cached = box.get(generationId);
    if (cached != null) {
      return jsonDecode(cached);
    }
    return null;
  }
  
  Future<void> updateCachedSection(
    String generationId,
    String sectionName,
    String content
  ) async {
    final cached = getCachedGeneration(generationId);
    if (cached != null) {
      cached['sections'] ??= {};
      cached['sections'][sectionName] = content;
      await cacheGeneration(generationId, cached);
    }
  }
  
  Future<void> clearCache() async {
    final box = Hive.box(GENERATIONS_BOX);
    await box.clear();
  }
}
```

### 8.2 Reconnection Logic

```dart
// lib/services/resilient_websocket_service.dart
import 'package:web_socket_channel/web_socket_channel.dart';
import 'dart:async';

class ResilientWebSocketService {
  WebSocketChannel? _channel;
  final StreamController<GenerationUpdate> _updateController = 
      StreamController<GenerationUpdate>.broadcast();
  
  Timer? _reconnectTimer;
  int _reconnectAttempts = 0;
  static const int MAX_RECONNECT_ATTEMPTS = 5;
  static const int RECONNECT_DELAY_MS = 2000;
  
  String? _currentGenerationId;
  bool _isConnected = false;
  
  Stream<GenerationUpdate> get updates => _updateController.stream;
  bool get isConnected => _isConnected;
  
  void connect(String generationId) {
    _currentGenerationId = generationId;
    _attemptConnection();
  }
  
  void _attemptConnection() {
    if (_reconnectAttempts >= MAX_RECONNECT_ATTEMPTS) {
      _updateController.addError('Max reconnection attempts reached');
      return;
    }
    
    try {
      final wsUrl = 'ws://your-backend.com/ws/generate/$_currentGenerationId';
      _channel = WebSocketChannel.connect(Uri.parse(wsUrl));
      
      _isConnected = true;
      _reconnectAttempts = 0;
      
      _channel!.stream.listen(
        (message) {
          final data = jsonDecode(message);
          
          if (data['type'] == 'heartbeat') {
            // Connection is alive
            return;
          }
          
          _updateController.add(GenerationUpdate.fromJson(data));
        },
        onError: (error) {
          _handleDisconnection();
        },
        onDone: () {
          _handleDisconnection();
        },
      );
    } catch (e) {
      _handleDisconnection();
    }
  }
  
  void _handleDisconnection() {
    _isConnected = false;
    _reconnectAttempts++;
    
    _updateController.add(GenerationUpdate(
      type: 'reconnecting',
      metadata: {'attempt': _reconnectAttempts}
    ));
    
    // Schedule reconnection
    _reconnectTimer?.cancel();
    _reconnectTimer = Timer(
      Duration(milliseconds: RECONNECT_DELAY_MS * _reconnectAttempts),
      _attemptConnection,
    );
  }
  
  void disconnect() {
    _reconnectTimer?.cancel();
    _channel?.sink.close();
    _isConnected = false;
  }
}
```

### 8.3 Progress Visualization

```dart
// lib/widgets/generation_progress_widget.dart
import 'package:flutter/material.dart';

class GenerationProgressWidget extends StatelessWidget {
  final int currentSection;
  final int totalSections;
  final String currentSectionName;
  final int tokensGenerated;
  
  const GenerationProgressWidget({
    required this.currentSection,
    required this.totalSections,
    required this.currentSectionName,
    required this.tokensGenerated,
  });
  
  @override
  Widget build(BuildContext context) {
    final progress = currentSection / totalSections;
    
    return Container(
      padding: EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              Text(
                'Generating: ${_formatSectionName(currentSectionName)}',
                style: Theme.of(context).textTheme.titleMedium,
              ),
              Text(
                '${currentSection}/${totalSections}',
                style: Theme.of(context).textTheme.bodyMedium,
              ),
            ],
          ),
          SizedBox(height: 12),
          LinearProgressIndicator(
            value: progress,
            minHeight: 8,
            backgroundColor: Colors.grey[300],
            valueColor: AlwaysStoppedAnimation<Color>(Colors.blue),
          ),
          SizedBox(height: 8),
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              Text(
                '${(progress * 100).toStringAsFixed(1)}% complete',
                style: Theme.of(context).textTheme.bodySmall,
              ),
              Text(
                '$tokensGenerated tokens',
                style: Theme.of(context).textTheme.bodySmall,
              ),
            ],
          ),
        ],
      ),
    );
  }
  
  String _formatSectionName(String name) {
    return name
        .split('_')
        .map((word) => word[0].toUpperCase() + word.substring(1))
        .join(' ');
  }
}
```

### 8.4 Error Handling UI

```dart
// lib/widgets/error_recovery_widget.dart
import 'package:flutter/material.dart';

class ErrorRecoveryWidget extends StatelessWidget {
  final String errorMessage;
  final String sectionName;
  final VoidCallback onRetry;
  final VoidCallback onSkip;
  
  const ErrorRecoveryWidget({
    required this.errorMessage,
    required this.sectionName,
    required this.onRetry,
    required this.onSkip,
  });
  
  @override
  Widget build(BuildContext context) {
    return Card(
      color: Colors.red[50],
      margin: EdgeInsets.all(16),
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              children: [
                Icon(Icons.error_outline, color: Colors.red),
                SizedBox(width: 8),
                Text(
                  'Error in ${_formatSectionName(sectionName)}',
                  style: TextStyle(
                    fontWeight: FontWeight.bold,
                    color: Colors.red[900],
                  ),
                ),
              ],
            ),
            SizedBox(height: 12),
            Text(
              errorMessage,
              style: TextStyle(color: Colors.red[800]),
            ),
            SizedBox(height: 16),
            Row(
              mainAxisAlignment: MainAxisAlignment.end,
              children: [
                TextButton(
                  onPressed: onSkip,
                  child: Text('Skip Section'),
                ),
                SizedBox(width: 8),
                ElevatedButton.icon(
                  onPressed: onRetry,
                  icon: Icon(Icons.refresh),
                  label: Text('Retry'),
                  style: ElevatedButton.styleFrom(
                    backgroundColor: Colors.red,
                    foregroundColor: Colors.white,
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
  
  String _formatSectionName(String name) {
    return name
        .split('_')
        .map((word) => word[0].toUpperCase() + word.substring(1))
        .join(' ');
  }
}
```

---

## 9. Production Checklist

### 9.1 Pre-Launch Checklist

- [ ] **Database**
  - [ ] Connection pooling configured
  - [ ] Indexes created for common queries
  - [ ] Backup strategy implemented
  - [ ] Migration scripts tested

- [ ] **Security**
  - [ ] JWT authentication implemented
  - [ ] API rate limiting active
  - [ ] CORS properly configured
  - [ ] Environment variables secured
  - [ ] SQL injection prevention verified

- [ ] **Monitoring**
  - [ ] Error tracking (Sentry) configured
  - [ ] Logging infrastructure ready
  - [ ] Performance metrics tracked
  - [ ] Health check endpoint active

- [ ] **Scalability**
  - [ ] Load balancer configured
  - [ ] Horizontal scaling tested
  - [ ] Redis cluster for high availability
  - [ ] Database read replicas considered

- [ ] **Testing**
  - [ ] Unit tests passing (>80% coverage)
  - [ ] Integration tests complete
  - [ ] Load testing performed
  - [ ] WebSocket reconnection tested

- [ ] **Documentation**
  - [ ] API documentation generated
  - [ ] README with setup instructions
  - [ ] Architecture diagrams created
  - [ ] Deployment guide written

### 9.2 Performance Optimization Tips

```python
# Example: Connection pooling optimization
engine = create_engine(
    DATABASE_URL,
    pool_size=20,              # Adjust based on load testing
    max_overflow=40,
    pool_timeout=30,
    pool_recycle=3600,
    pool_pre_ping=True,
    echo=False,                # Disable in production
    connect_args={
        "connect_timeout": 10,
        "options": "-c statement_timeout=30000"  # 30 second query timeout
    }
)
```

### 9.3 Monitoring Dashboard (Example with Grafana)

```yaml
# Example Prometheus metrics
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'fastapi'
    static_configs:
      - targets: ['backend:8000']
    metrics_path: /metrics
```

```python
# Add to main.py
from prometheus_fastapi_instrumentator import Instrumentator

instrumentator = Instrumentator()
instrumentator.instrument(app).expose(app)
```

---

## 10. Conclusion

This implementation guide provides a production-ready architecture for section-based streaming with Python and Flutter. Key takeaways:

1. **Section-based orchestration** enables reliable long-form content generation
2. **WebSocket streaming** provides real-time user feedback
3. **Proper error handling** and retry logic ensure robustness
4. **Database persistence** allows recovery from failures
5. **Redis caching** improves performance and enables resumption
6. **Flutter integration** delivers smooth cross-platform UX

### Next Steps

1. Start with the minimal implementation (FastAPI + PostgreSQL + WebSocket)
2. Add Redis for caching and session management
3. Implement Celery for background processing
4. Build Flutter UI with reconnection logic
5. Add monitoring and alerting
6. Scale horizontally as needed

The architecture is designed to scale from prototype to production while maintaining code quality and user experience.
