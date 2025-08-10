# Mata Talk - AI English Learning### 1. Voice Interaction Pipeline

#### English Learning Audio Flow
```
User Speech → Swift App → LiveKit → Python Agent
(English Practice) (Microphone) (WebRTC) (AI English Tutor)
``` Architecture

This repository contains both Python-based AI agent backend (`mata-talk-python`) and Swift-based client applications (`mata-talk-swift`) that work together to create a complete AI English learning conversation system using LiveKit's real-time communication platform.

## System Overview

The system consists of three main layers:

1. **Backend AI Agent** (Python) - The intelligent English learning conversation partner
2. **LiveKit Cloud/Server** - Real-time communication infrastructure  
3. **Frontend Client Apps** (Swift) - User interface for English conversation practice

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Swift Client  │◄──►│  LiveKit Cloud  │◄──►│  Python Agent   │
│   (iOS/macOS)   │    │   (WebRTC/WS)   │    │ (English Tutor) │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## Component Interaction Flow

### 1. Connection Establishment

**Swift Client (`AppViewModel`)**:
- `TokenService` requests authentication tokens from LiveKit Cloud sandbox
- `AppViewModel.connect()` establishes WebRTC connection to LiveKit room
- Configures audio/video tracks and device permissions

**Python Agent**:
- Monitors LiveKit room for new participants
- Automatically joins when a user connects
- Initializes AI pipeline (LLM, STT, TTS, VAD)

### 2. Voice Interaction Pipeline

#### Audio Capture & Processing
```
User Speech → Swift App → LiveKit → Python Agent
             (Microphone)  (WebRTC)   (STT/Deepgram)
```

**Swift Client Components**:
- `AudioManager` handles microphone input and device selection
- `LocalParticipantView` shows user's camera/audio visualization
- Audio tracks published to LiveKit room

**Python Agent Processing**:
- `silero.VAD` detects when user starts/stops speaking
- `deepgram.STT` converts speech to text
- `openai.LLM` processes text with English learning context
- `cartesia.TTS` converts educational response to speech
- `MultilingualModel` manages turn detection for natural conversation

#### Tutor Response & Learning Feedback
```
English Tutor Response ← Swift App ← LiveKit ← Python Agent
(Learning Feedback)    (Speakers)   (WebRTC)   (Educational AI)
```

### 3. Real-time Chat & Transcription

**Message Flow Architecture**:
```
┌──────────────────┐    ┌─────────────────┐    ┌───────────────────┐
│ ChatViewModel    │◄──►│ MessageReceiver │◄──►│ LiveKit Text      │
│ (UI State)       │    │ (Aggregation)   │    │ Streams           │
└──────────────────┘    └─────────────────┘    └───────────────────┘
         ▲                                              ▲
         │                                              │
┌──────────────────┐    ┌─────────────────┐             │
│ MessageSender    │───►│ LocalMessage    │─────────────┘
│ (User Input)     │    │ Sender          │
└──────────────────┘    └─────────────────┘
```

**Key Components**:

- **`ChatViewModel`**: Central coordinator for all chat messages
  - Aggregates messages from multiple receivers
  - Provides unified interface for UI
  - Manages message ordering and deduplication

- **`TranscriptionStreamReceiver`**: Processes real-time transcriptions
  - Handles partial message chunks from agent
  - Aggregates segments into complete messages
  - Manages message finalization and cleanup

- **`LocalMessageSender`**: Handles user text input
  - Converts user messages to LiveKit data messages
  - Provides local echo for immediate UI feedback

### 4. Video & Screen Sharing

**Video Components**:
- `LocalParticipantView`: User's camera preview with flip controls
- `AgentParticipantView`: Agent's avatar (if enabled) or audio visualizer
- `ScreenShareView`: Screen sharing preview
- `CameraCapturer`: Manages camera device selection and switching

**Layout Management**:
- `VoiceInteractionView`: Voice-focused layout with minimal UI
- `TextInteractionView`: Chat-focused layout with text input
- `VisionInteractionView`: visionOS-specific spatial layout

### 5. Dependency Injection & State Management

**`Dependencies` Container**:
```swift
@MainActor
final class Dependencies {
    lazy var room = Room()                    // LiveKit connection
    lazy var tokenService = TokenService()   // Authentication
    lazy var messageReceivers = [...]         // Chat receivers
    lazy var messageSenders = [...]           // Chat senders
    lazy var errorHandler = { ... }           // Error handling
}
```

**State Flow**:
```
AppViewModel (Root State)
├── Room State (Connection, Participants, Tracks)
├── Device State (Audio/Video devices)
├── Interaction Mode (Voice/Text)
└── Error State

ChatViewModel (Chat State)  
├── Message Collection (Ordered Dictionary)
├── Message Receivers (Transcription, etc.)
└── Message Senders (User input)
```

## Data Flow Patterns

### 1. Unidirectional Data Flow
- ViewModels expose read-only state via `@Observable`
- Actions flow through async methods
- State updates propagate automatically to UI

### 2. Reactive Streams
- WebRTC tracks as reactive data sources
- AsyncStream for message aggregation
- Combine for device state changes

### 3. Actor-based Concurrency
- `TranscriptionStreamReceiver` uses actor isolation
- Thread-safe message processing
- Async/await throughout the stack

## Error Handling & Resilience

**Error Propagation**:
```
Component Error → ErrorHandler → AppViewModel → UI Alert
```

**Connection Resilience**:
- Automatic reconnection via LiveKit SDK
- State reset on disconnection
- Graceful degradation when agent unavailable

## Platform-Specific Adaptations

### iOS/iPadOS
- Touch-optimized controls
- Haptic feedback for voice interactions
- Background audio support

### macOS
- Device selection menus for audio/video
- Window-based interactions
- Menu bar controls

### visionOS
- Spatial UI layout
- Glass background effects
- Hand tracking integration

## Configuration & Customization

### Agent Features (`AgentFeatures`)
```swift
struct AgentFeatures: OptionSet {
    static let voice = Self(rawValue: 1 << 0)   // Voice conversation practice
    static let text = Self(rawValue: 1 << 1)    // Text chat for learning
    static let video = Self(rawValue: 1 << 2)   // Video for pronunciation
}
```

### Python English Tutor Pipeline
```python
session = AgentSession(
    llm=openai.LLM(model="gpt-4o-mini"),           # Language model for tutoring
    stt=deepgram.STT(model="nova-3"),               # Speech-to-text for pronunciation
    tts=cartesia.TTS(voice="..."),                  # Text-to-speech for clear speaking
    turn_detection=MultilingualModel(),             # Natural conversation flow
    vad=silero.VAD.load(),                         # Voice activity detection
    preemptive_generation=True,                    # Responsive teaching
)
```

## Development & Deployment

### Local Development
1. Start Python agent: `python src/agent.py`
2. Configure LiveKit sandbox in Swift app
3. Build and run iOS/macOS app from Xcode

### Production Deployment
1. Deploy Python agent to cloud infrastructure
2. Configure production LiveKit server
3. Update `TokenService` for production authentication
4. Build and distribute client apps

## Security Considerations

- **Token Authentication**: Secure token generation on server
- **Room Isolation**: Each session gets unique room
- **Audio Privacy**: Local audio processing where possible
- **Data Encryption**: End-to-end encryption via WebRTC

This architecture provides a scalable, real-time voice AI system with rich client applications and powerful backend processing capabilities.
