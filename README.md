# flutter_ai_toolkit

> **Production-grade AI toolkit for Flutter & Dart** — Clean Architecture, Repository & Strategy patterns, async-first, `Result<T>` error handling, streaming-first APIs, multi-provider plugin architecture, and ready-made chat UI widgets.

[![pub.dev](https://img.shields.io/badge/pub-v1.0.0-blue.svg)](https://pub.dev/packages/flutter_ai_toolkit)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Dart SDK](https://img.shields.io/badge/Dart->=3.3.0-00B4AB.svg)](https://dart.dev)
[![Flutter](https://img.shields.io/badge/Flutter->=3.19.0-02569B.svg)](https://flutter.dev)


---

## 🔗 Official Release

* **GitHub Release:**
  [https://github.com/Brah-Timo/flutter-ai-toolkit/releases/tag/Release](https://github.com/Brah-Timo/flutter-ai-toolkit/releases/tag/Release)

* **Repository:**
  [https://github.com/Brah-Timo/flutter-ai-toolkit](https://github.com/Brah-Timo/flutter-ai-toolkit)

* **Issues & Feature Requests:**
  [https://github.com/Brah-Timo/flutter-ai-toolkit/issues](https://github.com/Brah-Timo/flutter-ai-toolkit/issues)

---


## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Providers](#providers)
- [Embeddings](#embeddings)
- [Vector Store](#vector-store)
- [RAG Pipeline](#rag-pipeline)
- [Prompts](#prompts)
- [Memory](#memory)
- [Tools & Tool Calling](#tools--tool-calling)
- [Agents](#agents)
- [Security](#security)
- [Streaming](#streaming)
- [UI Widgets](#ui-widgets)
- [Utils](#utils)
- [Testing](#testing)
- [Example App](#example-app)
- [Roadmap](#roadmap)
- [TIMSoft Integration](#timsoft-integration)
- [License](#license)

---

## Overview

`flutter_ai_toolkit` is a comprehensive, production-ready AI toolkit designed for Flutter and Dart applications. It provides a complete stack from raw LLM inference to high-level ReAct agents, including:

- **Multi-provider support**: OpenAI, Anthropic (Claude), Google Gemini, Ollama (local), Groq, OpenRouter — all behind a single unified `LlmProvider` interface.
- **RAG (Retrieval-Augmented Generation)**: Full pipeline with chunking, embedding, vector storage, semantic + hybrid retrieval, reranking, and citation management.
- **ReAct Agents**: Tool-calling agents with Thought → Action → Observation loops, planner agents, and multi-agent orchestration.
- **Streaming-first**: Every LLM call supports token streaming via Dart `Stream<Result<InferenceResponse>>`.
- **`Result<T>` everywhere**: No uncaught exceptions — all operations return `Result<T>` (Success/Failure).
- **Clean Architecture**: 7-layer architecture with dependency injection, isolate-based heavy compute, and hierarchical LRU+SQLite caching.
- **Ready-made UI**: `AIChatWidget`, `StreamingChatWidget`, `PromptEditor`, `TokenUsageChart`, `VectorSearchView` — drop in and go.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  App Layer (Flutter UI, Navigator)                          │
├─────────────────────────────────────────────────────────────┤
│  UI Layer (AIChatWidget, StreamingChatWidget, PromptEditor) │
├─────────────────────────────────────────────────────────────┤
│  Agents / Orchestration (ReActAgent, PlannerAgent,          │
│                           MultiAgent)                       │
├─────────────────────────────────────────────────────────────┤
│  RAG / Prompt Pipeline (RagPipeline, PromptPipeline,        │
│                          ContextBuilder, CitationManager)   │
├─────────────────────────────────────────────────────────────┤
│  Core Services (LlmEngine, EmbeddingService, ChatSession,   │
│                  ToolExecutor, MemoryStore)                  │
├─────────────────────────────────────────────────────────────┤
│  Providers / Plugins (OpenAI, Anthropic, Gemini, Ollama,    │
│                        Groq, OpenRouter, ProviderRegistry)  │
├─────────────────────────────────────────────────────────────┤
│  Infrastructure (SQLite, HNSW index, IsolatePool,           │
│                   ApiKeyVault, HttpClient, Logger)          │
└─────────────────────────────────────────────────────────────┘
```

---

## Features

| Category | Feature |
|---|---|
| **LLM** | Multi-provider, streaming, tool calling, function calling, model manager |
| **Embeddings** | Embedding service, vector normalisation, LRU cache, cosine/dot/euclidean similarity |
| **Vector Store** | In-memory + SQLite backends, HNSW index, metadata filtering, batch upsert |
| **RAG** | Overlap / recursive / semantic chunkers, TXT/MD/HTML/JSON loaders, semantic + hybrid (BM25+dense) retrieval, LLM reranker, citation manager |
| **Prompts** | Template system with `{{variable}}` substitution, role prompts, prompt pipeline |
| **Memory** | Conversation (sliding window), summary memory, vector memory |
| **Tools** | Type-safe tool schema, tool registry, async executor, built-in calculator / HTTP / file / search tools |
| **Agents** | ReAct (Thought→Action→Observation), PlannerAgent, MultiAgent with router |
| **Security** | `ApiKeyVault` (flutter_secure_storage), `EncryptedStorage` (AES-256) |
| **Streaming** | SSE decoder, `StreamEvent`, `StreamHandler`, `TokenStream` |
| **UI** | Chat bubble, typing indicator, message input, streaming chat widget, prompt editor, token usage chart, vector search view |
| **Utils** | LRU cache, retry helper, structured logger, async HTTP client |
| **DI** | `ServiceLocator` (get_it-style), `IsolatePool` for CPU-bound work |
| **Error handling** | `Result<T>`, `AiException`, `CancellationToken` |

---

## Installation

Add to `pubspec.yaml`:

```yaml
dependencies:
  flutter_ai_toolkit: ^1.0.0
```

Then:

```bash
flutter pub get
```

Import:

```dart
import 'package:flutter_ai_toolkit/flutter_ai_toolkit.dart';
```

---

## Quick Start

### 1. Register a provider

```dart
import 'package:flutter_ai_toolkit/flutter_ai_toolkit.dart';

void main() {
  // Register OpenAI as default provider
  ProviderRegistry.instance.register(
    'openai',
    OpenAiProvider(
      apiKey: 'sk-your-api-key',
      defaultModel: 'gpt-4o-mini',
    ),
  );

  runApp(const MyApp());
}
```

### 2. Send a message

```dart
final engine = LlmEngine(
  provider: ProviderRegistry.instance.getDefault().value!,
);

final request = InferenceRequest(
  messages: [
    ChatMessage.system('You are a helpful assistant.'),
    ChatMessage.user('What is the capital of France?'),
  ],
  temperature: 0.7,
);

final result = await engine.complete(request);
result.when(
  success: (response) => print(response.text),
  failure: (error) => print('Error: ${error.message}'),
);
```

### 3. Stream tokens

```dart
final stream = engine.stream(request);
await for (final chunk in stream) {
  if (chunk.isSuccess) {
    stdout.write(chunk.value!.text);
  }
}
```

### 4. Drop-in chat UI

```dart
StreamingChatWidget(
  engine: engine,
  systemPrompt: 'You are a helpful assistant.',
  placeholder: 'Type a message…',
)
```

---

## Providers

All providers implement `LlmProvider` — a unified interface for completion and streaming.

### OpenAI

```dart
OpenAiProvider(
  apiKey: 'sk-…',
  defaultModel: 'gpt-4o-mini',     // gpt-4o, gpt-3.5-turbo, …
  baseUrl: 'https://api.openai.com/v1', // override for proxies
  organization: 'org-…',           // optional
)
```

### Anthropic (Claude)

```dart
AnthropicProvider(
  apiKey: 'sk-ant-…',
  defaultModel: 'claude-3-5-haiku-20261022',
  maxTokens: 4096,
)
```

### Google Gemini

```dart
GeminiProvider(
  apiKey: 'AIza…',
  defaultModel: 'gemini-1.5-flash',
)
```

### Ollama (local / self-hosted)

```dart
OllamaProvider(
  baseUrl: 'http://localhost:11434',
  defaultModel: 'llama3.2',        // any model pulled in Ollama
)
```

### Groq

```dart
GroqProvider(
  apiKey: 'gsk_…',
  defaultModel: 'llama-3.1-70b-versatile',
)
```

### OpenRouter

```dart
OpenRouterProvider(
  apiKey: 'sk-or-…',
  defaultModel: 'openai/gpt-4o-mini',
  appName: 'MyApp',
)
```

### Provider Registry

```dart
final registry = ProviderRegistry.instance;
registry.register('openai', openAiProvider);
registry.register('local', ollamaProvider);
registry.setDefault('openai');

// Route by id
final result = await registry.complete(request, providerId: 'local');
```

---

## Embeddings

```dart
final service = EmbeddingService(
  model: OpenAiEmbeddingModel(apiKey: 'sk-…', model: 'text-embedding-3-small'),
  cache: EmbeddingCache(maxSize: 1000),
);

final result = await service.embed('Hello, world!');
result.when(
  success: (vector) {
    print('Dim: ${vector.dimension}');
    print('Mag: ${vector.magnitude}');
  },
  failure: (e) => print(e.message),
);

// Similarity
final sim = Similarity.cosine(vecA, vecB); // -1.0 … 1.0
```

---

## Vector Store

### In-memory

```dart
final store = MemoryVectorStore();

await store.upsert(VectorDocument(
  id: 'doc1',
  content: 'Flutter is awesome',
  vector: [0.1, 0.2, …],
  metadata: {'source': 'blog'},
));

final results = await store.similaritySearch(queryVec, topK: 5);
for (final r in results) {
  print('[${r.score.toStringAsFixed(3)}] ${r.document.content}');
}
```

### SQLite (persistent)

```dart
final store = SqliteVectorStore(dbPath: '/data/vectors.db');
await store.open();
// Same API as MemoryVectorStore
```

### Metadata filtering

```dart
final filter = MetadataFilter.and([
  MetadataFilter.equals('category', 'science'),
  MetadataFilter.contains('author', 'Smith'),
]);
final results = await store.similaritySearch(queryVec, topK: 10, filter: filter);
```

---

## RAG Pipeline

```dart
final pipeline = RagPipeline(
  retriever: SemanticRetriever(
    embeddingService: embService,
    vectorStore: store,
    topK: 5,
  ),
  engine: llmEngine,
  config: RagConfig(
    systemPrompt: 'Answer based on the provided context only.',
    maxContextTokens: 2000,
    includeCitations: true,
  ),
);

final answer = await pipeline.ask('What is HNSW?');
print(answer.text);
print(answer.citations);   // list of source references

// Streaming
await for (final chunk in pipeline.askStream('Explain RAG pipelines.')) {
  stdout.write(chunk.text);
}
```

### Chunkers

```dart
// Fixed-size with overlap
final chunker = OverlapChunker(chunkSize: 500, overlapSize: 50);

// Recursive (paragraph → sentence → word fallback)
final chunker = RecursiveChunker(maxChunkSize: 500);

// Semantic (clusters similar sentences together)
final chunker = SemanticChunker(embeddingService: embService, threshold: 0.85);
```

### Loaders

```dart
final loader = MarkdownLoader();
final doc = await loader.load('/path/to/file.md');

final htmlLoader = HtmlLoader(stripTags: true);
final jsonLoader = JsonLoader(textField: 'content');
```

### Hybrid Retrieval (BM25 + Dense)

```dart
final retriever = HybridRetriever(
  denseRetriever: SemanticRetriever(embeddingService: embService, vectorStore: store),
  alpha: 0.6,    // weight for dense (0=BM25 only, 1=dense only)
  rrfK: 60,      // Reciprocal Rank Fusion constant
);
```

---

## Prompts

```dart
final template = PromptTemplate(
  name: 'qa_template',
  template: 'You are a {{persona}}. Answer the following: {{question}}',
  variables: {'persona': 'helpful assistant'},
);

final rendered = template.render(overrides: {'question': 'What is Flutter?'});

// Role prompts
final system = RolePrompt.system('You are concise and factual.');
final user = RolePrompt.user('Explain {{topic}} in 3 sentences.')
    .withVariables({'topic': 'HNSW indexing'});

// Prompt pipeline
final pipeline = PromptPipeline(steps: [
  PromptPipelineStep(rolePrompt: system),
  PromptPipelineStep(rolePrompt: user),
]);
final messages = pipeline.build();  // List<ChatMessage>
```

---

## Memory

### Conversation Memory (sliding window)

```dart
final memory = ConversationMemory(maxMessages: 20);
memory.add(ChatMessage.user('Hello'));
memory.add(ChatMessage.assistant('Hi there!'));

// Pass to engine
final request = InferenceRequest(messages: memory.messages);
```

### Summary Memory

```dart
final memory = SummaryMemory(summaryThreshold: 10);
// When messages exceed threshold, call your summarizer and update:
memory.setSummary(await summarize(memory.messages));
```

### Vector Memory (semantic search over history)

```dart
final memory = VectorMemory(store: MemoryVectorStore());
await memory.addMessage(message, vector: await embed(message.content));
final relevant = await memory.searchSimilar(queryVec, topK: 3);
```

---

## Tools & Tool Calling

### Built-in tools

```dart
// Calculator – evaluates math expressions
CalculatorTool()  // 2+2, 3*7, 2^10, sqrt(16), …

// HTTP GET
HttpTool()        // fetch URL, returns body text

// File read/write (sandboxed)
FileTool()

// Web search stub (wire to real API)
SearchTool()
```

### Custom tool

```dart
class WeatherTool extends AiTool {
  @override
  ToolSchema get schema => ToolSchema(
    name: 'get_weather',
    description: 'Get current weather for a city',
    parameters: {
      'city': ToolParameter(
        type: ParameterType.string,
        description: 'City name',
        required: true,
      ),
    },
  );

  @override
  Future<ToolResult> execute(Map<String, dynamic> args) async {
    final city = args['city'] as String;
    // call real weather API…
    return ToolResult.success(
      callId: '',
      toolName: 'get_weather',
      output: 'Sunny, 24°C in $city',
    );
  }
}
```

### Registry & Executor

```dart
final registry = ToolRegistry();
registry.register(CalculatorTool());
registry.register(WeatherTool());

final executor = ToolExecutor(registry: registry);
final result = await executor.execute(
  ToolCall(id: 'c1', toolName: 'calculator', arguments: {'expression': '42*7'}),
);
print(result.output); // 2940
```

---

## Agents

### ReAct Agent

```dart
final agent = ReActAgent(
  engine: llmEngine,
  toolRegistry: registry,
  config: AgentConfig(
    systemPrompt: 'You are a helpful assistant with tools.',
    maxIterations: 10,
  ),
);

final result = await agent.run('What is 123 * 456?');
if (result.isSuccess) {
  print(result.value!.answer);
  print('Steps: ${result.value!.steps.length}');
}
```

### Planner Agent

```dart
final planner = PlannerAgent(
  engine: llmEngine,
  config: AgentConfig(systemPrompt: 'Plan tasks step by step.'),
);
final result = await planner.run('Write a research report on AI safety.');
```

### Multi-Agent

```dart
final multiAgent = MultiAgent(
  agents: {
    'math': mathReActAgent,
    'research': researchPlannerAgent,
    'code': codeReActAgent,
  },
  routerProvider: ProviderRegistry.instance.get('openai').value!,
);

final result = await multiAgent.run('Solve the travelling salesman problem for 5 cities.');
```

### Cancellation

```dart
final token = CancellationToken();
// Cancel after 10s
Future.delayed(const Duration(seconds: 10), token.cancel);

final result = await agent.run('Long task…', cancellationToken: token);
```

---

## Security

### API Key Vault

```dart
final vault = ApiKeyVault();
await vault.store('openai', 'sk-your-key');

final key = await vault.retrieve('openai');
// Uses flutter_secure_storage under the hood (Keychain / Keystore)
```

### Encrypted Storage

```dart
final storage = EncryptedStorage(key: 'my-32-byte-aes-key-1234567890ab');
await storage.write('sensitive_data', jsonEncode(payload));
final data = await storage.read('sensitive_data');
```

---

## Streaming

```dart
// SSE stream (for HTTP event-stream responses)
final decoder = SseDecoder();
final events = decoder.decode(httpResponseStream);

// StreamHandler with backpressure
final handler = StreamHandler<String>(
  onData: (token) => buffer.write(token),
  onError: (e) => logger.error(e),
  onDone: () => print('Stream complete'),
);
```

---

## UI Widgets

### StreamingChatWidget

```dart
StreamingChatWidget(
  engine: llmEngine,
  systemPrompt: 'You are a helpful assistant.',
  placeholder: 'Ask me anything…',
  enableCancellation: true,
  onTokenUsage: (usage) => print('Tokens: ${usage.totalTokens}'),
)
```

### AIChatWidget (blocking)

```dart
AIChatWidget(
  session: ChatSession(engine: llmEngine),
  placeholder: 'Type your message…',
)
```

### PromptEditor

```dart
PromptEditor(
  engine: llmEngine,
  initialTemplate: 'Translate "{{text}}" to {{language}}.',
  variables: {'text': 'Hello', 'language': 'French'},
)
```

### TokenUsageChart

```dart
TokenUsageChart(
  entries: [
    TokenUsageEntry(label: 'Chat', promptTokens: 150, completionTokens: 300),
    TokenUsageEntry(label: 'RAG', promptTokens: 500, completionTokens: 200),
  ],
)
```

### VectorSearchView

```dart
VectorSearchView(
  store: vectorStore,
  embeddingService: embService,
  topK: 5,
)
```

---

## Utils

### Logger

```dart
final logger = AiLogger(tag: 'MyService');
logger.debug('Initialising…');
logger.info('Ready');
logger.warn('Rate limit approaching');
logger.error('Request failed', exception: e, stackTrace: st);
```

### LRU Cache

```dart
final cache = LruCache<String, String>(capacity: 256);
cache.put('key', 'value');
final val = cache.get('key');
```

### Retry Helper

```dart
final result = await RetryHelper.retry(
  () => httpClient.get(url),
  maxAttempts: 3,
  delay: const Duration(seconds: 1),
  retryIf: (e) => e is TimeoutException,
);
```

---

## Testing

Run all tests:

```bash
flutter test
```

Run specific suites:

```bash
# Unit tests
flutter test test/unit/core/result_test.dart
flutter test test/unit/embeddings/embedding_test.dart
flutter test test/unit/vector_store/vector_store_test.dart
flutter test test/unit/llm/llm_test.dart
flutter test test/unit/rag/chunker_test.dart
flutter test test/unit/tools/tools_test.dart
flutter test test/unit/agents/agents_test.dart
flutter test test/unit/prompts/prompts_test.dart
flutter test test/unit/memory/memory_test.dart
flutter test test/unit/providers/providers_test.dart

# Integration tests
flutter test test/integration/
```

---

## Example App

The `example/` directory contains a full demo app showcasing all features:

```
example/
├── lib/
│   ├── main.dart              # App entry, routes, theme
│   └── screens/
│       ├── home_screen.dart   # Feature grid
│       ├── chat_screen.dart   # Streaming chat demo
│       ├── rag_screen.dart    # Document ingestion + semantic search
│       ├── agent_screen.dart  # ReAct agent with tool log
│       └── settings_screen.dart # Provider/model configuration
└── pubspec.yaml
```

Run the example:

```bash
cd example
flutter run
```

---

## Roadmap

| Version | Milestone |
|---|---|
| **v0.1** ✅ | Core scaffold, embeddings, vector store, RAG basics |
| **v0.2** ✅ | Providers (OpenAI, Anthropic, Gemini, Ollama, Groq, OpenRouter) |
| **v0.3** ✅ | Agents (ReAct, Planner, Multi-agent), Tools |
| **v0.4** ✅ | Prompts, Memory, Streaming, Security, UI widgets |
| **v1.0** ✅ | Full test coverage, example app, documentation |
| **v1.1** 🔜 | Local LLM via FFI (llama.cpp bindings) |
| **v1.2** 🔜 | Full HNSW implementation (Dart-native) |
| **v1.3** 🔜 | Tool sandboxing (isolate-based execution) |
| **v1.4** 🔜 | Multimodal support (vision, audio) |
| **v1.5** 🔜 | Fine-tuning and RLHF helpers |
| **v2.0** 🔜 | Multi-modal agents, voice interface |

---

## TIMSoft Integration

`flutter_ai_toolkit` is designed to integrate seamlessly with the **TIMSoft ecosystem**:

- **`timsoftdz_core`**: Reuses `Result<T>` and `SecureStorage` — zero duplication.
- **`timsoft_rdl_designer`**: AI-assisted report design via `PromptPipeline` + `RagPipeline`.
- **`timsoft_rdl_renderer`**: Real-time AI suggestions during report rendering.

```dart
// Example: AI-assisted report generation in timsoft_rdl_designer
final pipeline = RagPipeline(
  retriever: SemanticRetriever(
    embeddingService: embService,
    vectorStore: reportTemplateStore,
  ),
  engine: llmEngine,
  config: RagConfig(
    systemPrompt: 'You are an expert report designer. Suggest improvements.',
  ),
);

final suggestion = await pipeline.ask(
  'How should I structure a quarterly sales report for retail?',
);
```

---

## License

```
MIT License

Copyright (c) 2026 TIMSoft

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
```

---

*Built with ❤️ by TIMSoft — powering the next generation of AI-native Flutter applications.*
