# Ollama Terminal Assistant

A feature-rich command-line interface for interacting with Ollama AI models locally. This terminal-based assistant provides a ChatGPT-like experience with streaming responses, conversation history, and model management capabilities.

## Overview

This project implements a C++ terminal application that communicates with the Ollama API to provide an interactive AI assistant experience. The application features real-time streaming responses, colorized output, conversation management, and comprehensive model handling.

## Architecture and Frameworks

### Core Technologies

- **C++17**: Primary programming language with modern C++ features
- **libcurl**: HTTP client library for API communication with Ollama
- **nlohmann/json**: JSON parsing and serialization library
- **Standard Template Library (STL)**: For containers, memory management, and threading

### Key Components

#### 1. ColorUtils Class
- **Purpose**: Terminal color management and ANSI escape sequence handling
- **Features**: Cross-platform color support (Windows/Unix), color detection, text styling
- **Implementation**: Static utility class with platform-specific initialization

#### 2. StreamingOutput Class
- **Purpose**: Simulates typing effects for enhanced user experience
- **Features**: Character-by-character output with variable delays, typing cursor simulation
- **Timing Logic**: Contextual delays (longer pauses for punctuation, shorter for spaces)

#### 3. OllamaAssistant Class
- **Purpose**: Core API communication and conversation management
- **Features**: HTTP request handling, streaming response processing, conversation history
- **API Integration**: RESTful communication with Ollama's chat endpoint

#### 4. TerminalInterface Class
- **Purpose**: User interface and command handling
- **Features**: Command parsing, interactive menus, status monitoring
- **User Experience**: Intuitive command system with help documentation

## Dependencies

### Required Libraries

```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install libcurl4-openssl-dev nlohmann-json3-dev build-essential

# CentOS/RHEL/Fedora
sudo yum install libcurl-devel json-devel gcc-c++
# or for newer versions:
sudo dnf install libcurl-devel json-devel gcc-c++

# macOS (using Homebrew)
brew install curl nlohmann-json

# Arch Linux
sudo pacman -S curl nlohmann-json base-devel
```

### Ollama Installation

```bash
# Install Ollama
curl -fsSL https://ollama.ai/install.sh | sh

# Start Ollama service
ollama serve

# Pull required models
ollama pull llama3.2
ollama pull codellama
ollama pull mistral
```

## Compilation

### Basic Compilation
```bash
g++ -std=c++17 -o ollama_assistant main.cpp -lcurl
```

### Optimized Build
```bash
g++ -std=c++17 -O2 -o ollama_assistant main.cpp -lcurl -pthread
```

### Debug Build
```bash
g++ -std=c++17 -g -DDEBUG -o ollama_assistant main.cpp -lcurl -pthread
```

### Cross-Platform Considerations
```bash
# Windows (MinGW)
g++ -std=c++17 -o ollama_assistant.exe main.cpp -lcurl -lws2_32

# macOS
clang++ -std=c++17 -o ollama_assistant main.cpp -lcurl
```

## Usage

### Starting the Application

```bash
# Use default model (llama3.2)
./ollama_assistant

# Specify a different model
./ollama_assistant codellama
./ollama_assistant mistral
```

### Available Commands

#### System Commands
- `/help` - Display comprehensive help information
- `/quit` or `/exit` - Exit the application gracefully
- `/status` - Check Ollama connection and system status

#### Conversation Management
- `/clear` - Clear conversation history and start fresh
- `/history` - Display entire conversation history
- `/stream` - Toggle streaming output mode on/off

#### Model Management
- `/models` - List all available installed models
- `/model` - Interactive model selection interface

### Command Examples

```bash
# Check if Ollama is running
/status

# List available models
/models

# Change to a different model
/model
# Then select from the numbered list

# Clear conversation and start over
/clear

# View conversation history
/history

# Toggle streaming mode
/stream
```

## Implementation Details

### HTTP Communication

The application uses libcurl for HTTP communication with the Ollama API:

```cpp
// API endpoint configuration
api_url = "http://localhost:11434/api/chat"

// Request headers
Content-Type: application/json

// JSON payload structure
{
    "model": "model_name",
    "messages": [conversation_history],
    "stream": true/false
}
```

### Conversation Management

Conversations are maintained as a JSON array:

```json
[
    {"role": "system", "content": "system_prompt"},
    {"role": "user", "content": "user_message"},
    {"role": "assistant", "content": "assistant_response"}
]
```

### Streaming Implementation

The application processes Server-Sent Events (SSE) for real-time responses:

```cpp
// Streaming response parsing
while (std::getline(stream, line)) {
    if (line.rfind("data: ", 0) == 0) {
        // Parse JSON chunk and extract content
        json chunk = json::parse(line.substr(6));
        std::string content = chunk["message"]["content"];
        // Display with typing effect
    }
}
```

### Color System

Cross-platform terminal color support:

```cpp
// ANSI color codes
"\033[31m" // Red
"\033[32m" // Green
"\033[34m" // Blue

// Windows compatibility
ENABLE_VIRTUAL_TERMINAL_PROCESSING
```

## Features

### Core Functionality
- Real-time streaming responses with typing effects
- Persistent conversation history within sessions
- Multiple model support and switching
- Connection status monitoring
- Cross-platform terminal color support

### User Experience Enhancements
- Interactive command system with tab completion hints
- Visual feedback for system operations
- Contextual help and error messages
- Graceful error handling and recovery

### Performance Optimizations
- Efficient JSON parsing and serialization
- Minimal memory footprint for conversation storage
- Optimized HTTP request handling
- Responsive UI with proper threading

## Troubleshooting

### Common Issues

#### Ollama Connection Failed
```bash
# Ensure Ollama is running
ollama serve

# Check if service is active
curl http://localhost:11434/api/tags
```

#### Model Not Found
```bash
# Install the model
ollama pull llama3.2

# Verify installation
ollama list
```

#### Compilation Errors
```bash
# Install missing dependencies
sudo apt-get install libcurl4-openssl-dev nlohmann-json3-dev

# Check compiler version
g++ --version  # Should support C++17
```

#### Color Display Issues
```bash
# Force color output (if supported)
export TERM=xterm-256color

# Disable colors if problematic
# Colors are auto-detected based on terminal capabilities
```

## Development

### Code Structure
```
main.cpp
├── ColorUtils class      # Terminal color management
├── StreamingOutput class # Typing effects and output
├── OllamaAssistant class # API communication
├── TerminalInterface class # User interface
└── main() function       # Application entry point
```

### Extension Points
- Add new commands by extending `handleCommand()` method
- Implement additional models by modifying model management
- Enhance streaming with custom formatting
- Add configuration file support

### Testing
```bash
# Test Ollama connection
curl -X POST http://localhost:11434/api/chat \
  -H "Content-Type: application/json" \
  -d '{"model":"llama3.2","messages":[{"role":"user","content":"Hello"}]}'

# Test model availability
curl http://localhost:11434/api/tags
```

## Security Considerations

- All communication occurs locally (localhost:11434)
- No external API keys or authentication required
- Conversation history stored in memory only
- No persistent data storage or logging

## Performance

- **Memory Usage**: Minimal, scales with conversation length
- **CPU Usage**: Low, mainly during model inference
- **Network**: Local HTTP requests only
- **Startup Time**: < 1 second on modern systems

