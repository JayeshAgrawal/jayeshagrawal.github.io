---
title: "Building Reliable RAG Systems in .NET"
author: "Jayesh Agrawal"
date: 2025-12-20 20:55:00 +0530
categories: [aiagent, dotnet]
tags: [microsoftagentframework, ragsystems]
seo:
  date_modified: 2021-02-08 01:55:41 +0530
---
# Building Reliable RAG Systems in .NET: From Naive Retrieval to Engineer-Grade Pipelines
---

## Introduction: (How to Build It Right)
We've probably seen the hype: "RAG is the solution to hallucinations. Just add our documents and our AI becomes reliable."

Then we built it. And it failed.

Our retrieval surface was too broad—returning irrelevant chunks. Or too narrow—missing critical context. Our embeddings couldn't distinguish between "refund policy for digital goods" and "refund policy for services." The LLM rambled despite having perfect documentation. Our costs exploded because we embedded every document, every query, every variation.

They ask: "How do I add documents to my LLM?" instead of: "How do I build a retrieval pipeline that:
- Serves queries in 50–200 ms?
- Maintains >70% precision (fewer false positives)?
- Costs <0.01 USD per query (or 0 with local models)?
- Handles 50 different document types correctly?
- Catches hallucinations before they reach users?
- Runs entirely on-premise if needed?"

**Lets find answers those questions.**

RAG is treated as infrastructure: with measurable SLAs, domain-specific chunking strategies, evaluation frameworks, and code we can run today. Five architecture patterns help we pick the right one upfront—not after months of painful refactoring. The code is extensible: it runs with local models (zero cost, complete privacy) or remote LLMs (maximum accuracy, managed infrastructure) via configuration, not rewrites. A complete, evaluated end-to-end example ties it together.

By the end, we will have:
- A chunking strategy for Our document types
- A retrieval pattern optimized for Our constraints
- Evaluation metrics that prove it is working
- Semantic Kernel + local SLM and/or remote LLM
- A deployment checklist we can actually ship with

---

## What we covers
- **5 architecture patterns**: From simple FAQ-style RAG to iterative agent verification
- **Domain-specific chunking**: Technical docs, legal clauses, code, FAQs, structured/time-series
- **Local vs remote models**: Build once, run with:
  - Local: Ollama, ONNX (0 USD, on-prem)
  - Remote: Azure OpenAI (managed, high accuracy)
- **Extensible design**: Interfaces for embeddings and LLMs; Semantic Kernel-based implementations
- **Evaluation framework**: Test cases, precision/recall, category breakdowns, regression checks
- **Examples**: An extensible `ExtensibleDocumentRAGSystem` that handles ingestion, retrieval, generation, and metrics

---

## The Challenge: RAG is Easy. Reliable RAG is Hard.

Naive RAG looks like this:

1. Split documents at fixed 512-token boundaries  
2. Embed everything with a generic model  
3. Retrieve top‑5 chunks  
4. Feed to LLM  
5. Hope it works

What actually happens:
- **Splitting destroys structure**: Functions separate from docstrings; contract clauses lose references; FAQs split Q from A.
- **Embeddings mismatch domain**: Legal, medical, and code need different behavior than general semantic search.
- **Retrieval lacks recall**: The relevant chunk is ranked 7th, but we only fetch 5. Or we get five near-duplicate chunks.
- **Generation hallucinates**: The LLM fills gaps instead of saying "I don't know."
- **Costs explode**: We increase k to compensate, tokens increase, latency and bills follow.

Reliable RAG requires engineering discipline:
1. **Parse, don't blind-split**: Respect document structure.
2. **Model selection based on task and constraints**: Local vs remote, speed vs quality.
3. **Measure retrieval independently**: Before calling an LLM, know Our retrieval P@5 and R@5.
4. **Make it observable**: Latency, cost, and quality metrics for each layer.
5. **Keep backends pluggable**: Local SLMs for development/on-prem; remote LLMs when needed—without invasive changes.

---

## A Three-Layer Mental Model

Think of RAG as three independent, measurable layers:

![image.png](/assets/img/posts/mental-model.jpg)

```
Layer 1: Knowledge Preparation (Ingestion)
├─ Parse documents (understand structure)
├─ Chunk at semantic boundaries
├─ Embed chunks (local SLM or remote API)
└─ Store in vector database

Layer 2: Query Processing (Retrieval)
├─ Embed query (same service as Layer 1)
├─ Search vector store (+ optional BM25)
├─ Rank results (optional re-ranking)
└─ Format context for Layer 3

Layer 3: Generation (Answer Production)
├─ Take query + retrieved context
├─ Call LLM (local or remote)
├─ Stream response
└─ Track cost + latency
```

Each layer has SLAs (latency, accuracy, cost) and pluggable backends. Lets implement this in Semantic Kernel, local SLMs, and remote LLMs.

---

## 1. What RAG Actually Is: An Engineering Definition

At its core, RAG is **a typed data pipeline we own, operate, and optimize**.

**Pipeline shape:**
- Input: Query string + optional context (user ID, filters, metadata)
- Stage 1 (Embedding): Convert query to a vector (local SLM or remote API)
- Stage 2 (Retrieval): Find top‑k most similar document vectors
- Stage 3 (Ranking): Reorder based on additional relevance signals
- Stage 4 (Context Preparation): Format chunks for an LLM prompt
- Output: Structured context (chunks, scores, source IDs, confidence) for generation

When treated as infrastructure, we ask:
- What is acceptable latency: 50 ms, 200 ms, 1 s?
- What precision/recall are required?
- What is the max acceptable cost per query?
- How are embedding/LLM model changes validated?
- How is retrieval correctness tested?

Lets answers these from an engineering perspective.

---

## 2. Five Architecture Patterns for .NET RAG Systems

Most teams build only a "sequential" pattern. That works for trivial cases but is suboptimal elsewhere. These five patterns let we pick intentionally.

### Pattern 1: Simple Sequential RAG (Baseline)

```
Query → Embed (Local SLM) → Vector Search → Top‑k Chunks → LLM
```

**When to use:**
- Corpus < 100k documents
- Simple, single-intent queries
- Low accuracy bar: "good enough"
- On-premise / cost-sensitive

**Pros:**  
Simple, low latency, fully local.  
**Cons:**  
No hybrid search, struggles with very keyword-heavy domains.

**Semantic Kernel + Local SLM Example**

```csharp
public class LocalRagPattern
{
    private readonly Kernel _kernel;
    private readonly ITextEmbeddingGenerationService _embedding;

    public LocalRagPattern()
    {
        var builder = Kernel.CreateBuilder();

        builder.AddOllamaTextEmbedding(
            modelId: "nomic-embed-text",
            endpoint: new Uri("http://localhost:11434"));

        _kernel = builder.Build();
        _embedding = _kernel.GetRequiredService<ITextEmbeddingGenerationService>();
    }

    public async Task<List<RetrievalResult>> RetrieveSimple(
        string query,
        VectorStore vectorStore)
    {
        var queryEmbedding = await _embedding.GenerateEmbeddingAsync(query);
        var results = vectorStore.Search(queryEmbedding, k: 5);
        return results;
    }
}
```

---

### Pattern 2: Hybrid Retrieval Pipeline

```
Query → [BM25 Branch + Dense Embedding (Local/Remote)] → Score Fusion → Rerank → LLM
```

**When to use:**
- Heavy domain-specific terminology (legal/medical/technical)
- Mix of "ID-style" queries and semantic ones
- Latency 50–200 ms is acceptable

**Pros:**  
Combines exact keyword matches and semantics.  
**Cons:**  
More moving parts (BM25 + vector store + fusion).

Implement via Reciprocal Rank Fusion (RRF):

```csharp
public class HybridRetriever
{
    private readonly IVectorStore _vectorStore;
    private readonly IBm25Index _bm25;
    private readonly IEmbeddingService _embedding;

    public async Task<List<RetrievalResult>> RetrieveHybrid(
        string query, 
        int k = 10)
    {
        var queryEmbedding = await _embedding.EmbedTextAsync(query);
        var denseResults = await _vectorStore.SearchAsync(queryEmbedding, k);
        var bm25Results = _bm25.Search(query, k);

        // RRF: Normalize and fuse scores
        var fused = new Dictionary<string, double>();

        foreach (var (idx, result) in denseResults.Select((r, i) => (i, r)))
        {
            if (!fused.ContainsKey(result.ChunkId))
                fused[result.ChunkId] = 0;
            fused[result.ChunkId] += 1.0 / (60 + idx + 1);
        }

        foreach (var (idx, result) in bm25Results.Select((r, i) => (i, r)))
        {
            if (!fused.ContainsKey(result.ChunkId))
                fused[result.ChunkId] = 0;
            fused[result.ChunkId] += 1.0 / (60 + idx + 1);
        }

        return fused
            .OrderByDescending(x => x.Value)
            .Take(k)
            .Select(x => new RetrievalResult { ChunkId = x.Key, Score = x.Value })
            .ToList();
    }
}
```

---

### Pattern 3: Multi-Index Strategy with Agent Routing

```
Query → Agent (Semantic Kernel) → Classifier Plugin → Target Index → Retrieve → LLM
```

**When to use:**
- Heterogeneous corpora (API docs, FAQs, legal, community posts)
- Need index-specific chunking and tuning
- >500k docs across types

**Pros:**  
Each index is optimized independently; improved precision.  
**Cons:**  
Requires an agent classifier and multiple indexes.

**Semantic Kernel - Agent Routing Example**

```csharp
public class MultiIndexAgentPattern
{
    private readonly Kernel _kernel;

    public MultiIndexAgentPattern()
    {
        var builder = Kernel.CreateBuilder();

        builder.AddOllamaTextEmbedding("nomic-embed-text",
            new Uri("http://localhost:11434"));
        builder.AddOllamaChatCompletion("mistral",
            new Uri("http://localhost:11434"));

        _kernel = builder.Build();

        _kernel.ImportPluginFromType<DocumentationRetriever>("docs");
        _kernel.ImportPluginFromType<LegalDocRetriever>("legal");
        _kernel.ImportPluginFromType<FaqRetriever>("faq");
    }

    public async Task<string> RetrieveWithAgentRouting(string query)
    {
        var functionPrompt = @"
Given the user query, determine which document type to search:
- ""API"": For authentication, endpoints, integration questions
- ""LEGAL"": For terms, compliance, policy questions
- ""FAQ"": For common questions, troubleshooting
- ""DOCS"": For general product documentation

Query: {{$query}}

Return ONLY the category name (API/LEGAL/FAQ/DOCS).";

        var result = await _kernel.InvokePromptAsync(
            functionPrompt,
            new KernelArguments { ["query"] = query });

        var category = result.ToString().Trim();

        return category switch
        {
            "API" => await _kernel.InvokeAsync<string>("docs", "RetrieveAPI",
                new KernelArguments { ["query"] = query }),
            "LEGAL" => await _kernel.InvokeAsync<string>("legal", "RetrieveLegal",
                new KernelArguments { ["query"] = query }),
            "FAQ" => await _kernel.InvokeAsync<string>("faq", "RetrieveFAQ",
                new KernelArguments { ["query"] = query }),
            _ => await _kernel.InvokeAsync<string>("docs", "RetrieveDocs",
                new KernelArguments { ["query"] = query })
        };
    }
}

public class DocumentationRetriever
{
    [KernelFunction("RetrieveAPI")]
    public string RetrieveAPI(string query)
    {
        return "Retrieved API docs...";
    }
}
```

---

### Pattern 4: Adaptive Retrieval (Query-Aware Top‑K)

```
Query → Complexity Analysis (Local SLM or heuristics) → Select k → Retrieve → LLM
```

**When to use:**
- High query volume
- Wide mix of query complexity
- Cost or latency sensitive

**Pros:**  
Big savings on tokens and retrieval without hurting accuracy.  
**Cons:**  
Complexity classifier must not be too expensive.

**Semantic Kernel + Local SLM Classifier**

```csharp
public class AdaptiveRetrievalPattern
{
    private readonly Kernel _kernel;

    public AdaptiveRetrievalPattern()
    {
        var builder = Kernel.CreateBuilder();
        builder.AddOllamaTextEmbedding("nomic-embed-text",
            new Uri("http://localhost:11434"));
        builder.AddOllamaChatCompletion("mistral",
            new Uri("http://localhost:11434"));

        _kernel = builder.Build();
    }

    public async Task<List<Chunk>> RetrieveAdaptive(string query, VectorStore vectorStore)
    {
        var prompt = @"
Analyze query complexity (1-5):
1 = Single word lookup
3 = Simple question
5 = Complex comparison with conditions

Query: {{$query}}

Return ONLY the number.";

        var result = await _kernel.InvokePromptAsync(
            prompt,
            new KernelArguments { ["query"] = query });

        int complexity = int.Parse(result.ToString().Trim());

        int k = complexity switch
        {
            1 => 2,
            2 => 3,
            3 => 5,
            4 => 8,
            5 => 12,
            _ => 5
        };

        var embedding = await _kernel
            .GetRequiredService<ITextEmbeddingGenerationService>()
            .GenerateEmbeddingAsync(query);

        var results = vectorStore.Search(embedding, k);

        Console.WriteLine($"Query complexity: {complexity}/5, Retrieved k={k} chunks");

        return results;
    }
}
```

---

### Pattern 5: Iterative Refinement RAG with Verification

```
Query → Retrieve → Draft Answer → Confidence Score → (Optional) Refined Query → Retrieve → Final Answer
```

**When to use:**
- High-stakes use (medical, legal, compliance)
- Need explicit verification passes
- Accept higher latency

**Semantic Kernel Implementation**

```csharp
public class IterativeRefinementPattern
{
    private readonly Kernel _kernel;

    public IterativeRefinementPattern()
    {
        var builder = Kernel.CreateBuilder();
        builder.AddOllamaTextEmbedding("nomic-embed-text",
            new Uri("http://localhost:11434"));
        builder.AddOllamaChatCompletion("mistral",
            new Uri("http://localhost:11434"));

        _kernel = builder.Build();
    }

    public async Task<(string answer, decimal confidence, int iterations)>
        RetrieveWithIterativeRefinement(string query, VectorStore vectorStore)
    {
        int iterations = 0;
        decimal confidence = 0;
        string answer = "";
        var refinedQuery = query;

        while (iterations < 3 && confidence < 0.75m)
        {
            iterations++;

            var embedding = await _kernel
                .GetRequiredService<ITextEmbeddingGenerationService>()
                .GenerateEmbeddingAsync(refinedQuery);

            var chunks = vectorStore.Search(embedding, k: 5);
            var context = string.Join("\n", chunks.Select(c => c.Text));

            var generatePrompt = $@"
                    Based on this context, answer the question.
                    Context: {context}
                    Question: {refinedQuery}

                    Provide a concise answer.";

            answer = await _kernel.InvokePromptAsync(generatePrompt);

            var scorePrompt = $@"
                    Rate Our confidence in this answer (0.0-1.0):
                    Answer: {answer}
                    Context quality: {(chunks.Count >= 3 ? "Good" : "Limited")}

                    Return ONLY a decimal (e.g., 0.85).";

            var scoreResult = await _kernel.InvokePromptAsync(scorePrompt);
            confidence = decimal.Parse(scoreResult.ToString().Trim());

            Console.WriteLine($"Iteration {iterations}: Confidence = {confidence:P}");

            if (confidence < 0.75m && iterations < 3)
            {
                var refinePrompt = $@"
                    The answer has low confidence. What aspect should we search for more deeply?
                    Original query: {query}
                    Answer: {answer}
                    Context: {context}

                    Suggest a refined search query to verify or improve this answer.
                    Return ONLY the new query.";

                refinedQuery = await _kernel.InvokePromptAsync(refinePrompt);
                Console.WriteLine($"Refined query: {refinedQuery}");
            }
        }

        return (answer, confidence, iterations);
    }
}
```

---

## 3. Domain-Specific Chunking: Beyond Generic Splits

Chunking is where most RAG systems lose points. Generic 512-token splits work fine for general text but fail for structured content.

### Chunking for Technical Documentation

Documentation has structure: headers, code blocks, explanations.

**Problem with generic chunking:** Code example gets split from its explanation. Header loses context.

**Smart approach:**
- Parse the document structure (markdown or HTML)
- Group content by heading level
- Keep code blocks + explanations together
- Include breadcrumb metadata

**C# Pattern:**

```csharp
public class DocumentationChunker
{
    public List<Chunk> ChunkMarkdownDocument(string markdown)
    {
        var lines = markdown.Split('\n');
        var chunks = new List<Chunk>();
        var currentChunk = new StringBuilder();
        var breadcrumb = new Stack<string>();

        foreach (var line in lines)
        {
            if (line.StartsWith("# "))
                breadcrumb.Clear();
            else if (line.StartsWith("## "))
                breadcrumb.Push(line.Substring(2));
            else if (line.StartsWith("### "))
                breadcrumb.Push(line.Substring(3));

            currentChunk.AppendLine(line);

            if ((line.StartsWith("## ") || line.StartsWith("# ")) && currentChunk.Length > 0)
            {
                chunks.Add(new Chunk
                {
                    Text = currentChunk.ToString(),
                    Metadata = new Dictionary<string, string>
                    {
                        { "breadcrumb", string.Join(" > ", breadcrumb.Reverse()) },
                        { "doc_type", "technical_docs" },
                        { "source_file", "api_reference.md" }
                    },
                    TokenCount = CountTokens(currentChunk.ToString())
                });
                currentChunk.Clear();
            }
        }

        if (currentChunk.Length > 0)
        {
            chunks.Add(new Chunk
            {
                Text = currentChunk.ToString(),
                Metadata = new Dictionary<string, string>
                {
                    { "breadcrumb", string.Join(" > ", breadcrumb.Reverse()) }
                }
            });
        }

        return chunks;
    }

    private int CountTokens(string text) => text.Length / 4;
}
```

**Expected behavior:**
- Input: API reference with 50 endpoints
- Output: ~50 chunks, one per endpoint + description
- Retrieval: "How do I authenticate?" finds the auth section, complete with examples

---

### Chunking for Legal / Compliance Documents

Legal documents have clause numbers, cross-references, and defined terms that matter.

**Problem with generic chunking:** Loses clause structure. Cross-references ("See Section 5.2") become useless.

**Smart approach:**
- Preserve clause numbers and section boundaries
- Add metadata for all cross-references
- Keep defined terms together
- Track version/date

**C# Pattern:**

```csharp
public class LegalDocumentChunker
{
    public List<Chunk> ChunkLegalDocument(string docText)
    {
        var clauses = ParseClauses(docText);
        var chunks = new List<Chunk>();
        var references = ExtractReferences(docText);

        foreach (var clause in clauses)
        {
            var clauseNumber = clause.Number;
            var text = clause.Text;

            var relatedReferences = references
                .Where(r => r.MentionedIn.Contains(clauseNumber))
                .Select(r => r.TargetClause)
                .ToList();

            chunks.Add(new Chunk
            {
                Text = text,
                Metadata = new Dictionary<string, string>
                {
                    { "clause_number", clauseNumber },
                    { "cross_references", string.Join(", ", relatedReferences) },
                    { "doc_type", "legal" },
                    { "version", "2.1" },
                    { "defined_terms", ExtractDefinedTerms(text) }
                }
            });
        }

        return chunks;
    }
}
```

---

### Chunking for Code + Docstrings

Code needs context: function signature, docstring, usage example.

**Problem:** If we split at arbitrary boundaries, we separate the function from its documentation.

**Smart approach:**
- Parse at language-aware boundaries (functions, classes, methods)
- Keep docstring + implementation + example together
- Include language/framework metadata

**Example:**

```csharp
public class CodeDocumentationChunker
{
    public List<Chunk> ChunkCSharpCode(string codeText)
    {
        var methods = ParseMethods(codeText);
        var chunks = new List<Chunk>();

        foreach (var method in methods)
        {
            var chunk = new StringBuilder();

            if (!string.IsNullOrEmpty(method.Docstring))
                chunk.AppendLine(method.Docstring);

            chunk.AppendLine(method.Signature);

            var implementation = method.Body
                .Split('\n')
                .Take(50)
                .ToList();
            chunk.AppendLine(string.Join("\n", implementation));

            chunks.Add(new Chunk
            {
                Text = chunk.ToString(),
                Metadata = new Dictionary<string, string>
                {
                    { "type", "method" },
                    { "name", method.Name },
                    { "class", method.ParentClass },
                    { "returns", method.ReturnType },
                    { "language", "csharp" },
                    { "file", method.FileName }
                }
            });
        }

        return chunks;
    }
}
```

---

### Chunking for FAQ / Q&A Pairs

FAQs should never split Q from A.

**Problem:** Generic chunking might put "Q: How do I reset my password?" in one chunk and the answer in another.

**Smart approach:**
- Keep Q&A pairs together
- Add question as metadata (for keyword search)
- Include context tags (e.g., "topic: authentication")

**Example:**

```csharp
public class FaqChunker
{
    public List<Chunk> ChunkFaq(string faqText)
    {
        var qaRegex = new Regex(@"Q:\s*(.+?)\s*A:\s*(.+?)(?=Q:|$)", 
            RegexOptions.Singleline);
        
        var matches = qaRegex.Matches(faqText);
        var chunks = new List<Chunk>();

        foreach (Match match in matches)
        {
            var question = match.Groups[1].Value.Trim();
            var answer = match.Groups[2].Value.Trim();

            chunks.Add(new Chunk
            {
                Text = $"Q: {question}\n\nA: {answer}",
                Metadata = new Dictionary<string, string>
                {
                    { "question", question },
                    { "doc_type", "faq" },
                    { "topic", InferTopic(question) },
                    { "difficulty", "beginner" }
                }
            });
        }

        return chunks;
    }

    private string InferTopic(string question)
    {
        if (question.Contains("refund") || question.Contains("payment"))
            return "billing";
        if (question.Contains("authenticate") || question.Contains("login"))
            return "authentication";
        return "general";
    }
}
```

---

## 4. Local SLM Embedding Options for .NET

Instead of relying on expensive API-based embeddings, use local Small Language Models (SLMs) running in .NET.

### Option 1: Ollama (Recommended for Development)

**Setup:**
```bash
ollama pull nomic-embed-text
# Runs on http://localhost:11434
```

**Semantic Kernel Integration:**

```csharp
public class OllamaEmbeddingService : IEmbeddingService
{
    private readonly Kernel _kernel;
    private readonly ITextEmbeddingGenerationService _service;

    public OllamaEmbeddingService(Uri ollamaEndpoint)
    {
        var builder = Kernel.CreateBuilder();
        builder.AddOllamaTextEmbedding("nomic-embed-text", ollamaEndpoint);

        _kernel = builder.Build();
        _service = _kernel.GetRequiredService<ITextEmbeddingGenerationService>();
    }

    public async Task<float[]> EmbedTextAsync(string text)
    {
        var embedding = await _service.GenerateEmbeddingAsync(text);
        return embedding.ToArray();
    }

    public async Task<List<float[]>> EmbedBatchAsync(List<string> texts)
    {
        var list = new List<float[]>();
        foreach (var text in texts)
            list.Add(await EmbedTextAsync(text));
        return list;
    }

    public string GetBackendName() => "Ollama (Local)";
}
```

**Performance:** ~100ms per embedding locally, fully offline.

**Cost:** 0 USD (runs on Our hardware).

---

### Option 2: ONNX Runtime (Production)

For production deployments, use ONNX Runtime to run embeddings directly in Our .NET application without external services.

```csharp
public class OnnxEmbeddingService : IEmbeddingService
{
    private readonly InferenceSession _session;

    public OnnxEmbeddingService(string onnxModelPath)
    {
        _session = new InferenceSession(onnxModelPath);
    }

    public float[] EmbedText(string text)
    {
        var tokens = Tokenize(text);

        var input = new List<NamedOnnxValue>
        {
            NamedOnnxValue.CreateFromTensor("input_ids",
                new DenseTensor<long>(tokens, new[] { 1, tokens.Length }))
        };

        using (var results = _session.Run(input))
        {
            return results.FirstOrDefault()?
                .AsEnumerable<float>()
                .ToArray() ?? new float[0];
        }
    }

    public async Task<float[]> EmbedTextAsync(string text)
    {
        return await Task.FromResult(EmbedText(text));
    }

    public async Task<List<float[]>> EmbedBatchAsync(List<string> texts)
    {
        var list = new List<float[]>();
        foreach (var text in texts)
            list.Add(EmbedText(text));
        return await Task.FromResult(list);
    }

    private long[] Tokenize(string text)
    {
        return new long[] { /* token ids */ };
    }

    public string GetBackendName() => "ONNX Runtime (Local, In-Process)";
}
```

**Performance:** ~20–50ms per embedding (very fast, in-process).

**Cost:** 0 USD, completely offline.

---

### Option 3: Azure OpenAI (When Local SLMs Aren't Enough)

For maximum accuracy, use Azure OpenAI while maintaining control:

```csharp
public class AzureOpenAIEmbeddingService : IEmbeddingService
{
    private readonly Kernel _kernel;
    private readonly ITextEmbeddingGenerationService _service;

    public AzureOpenAIEmbeddingService(string deploymentName, Uri endpoint, string apiKey)
    {
        var builder = Kernel.CreateBuilder();
        builder.AddAzureOpenAITextEmbedding(deploymentName, endpoint, apiKey);

        _kernel = builder.Build();
        _service = _kernel.GetRequiredService<ITextEmbeddingGenerationService>();
    }

    public async Task<float[]> EmbedTextAsync(string text)
    {
        var embedding = await _service.GenerateEmbeddingAsync(text);
        return embedding.ToArray();
    }

    public async Task<List<float[]>> EmbedBatchAsync(List<string> texts)
    {
        var list = new List<float[]>();
        foreach (var text in texts)
            list.Add(await EmbedTextAsync(text));
        return list;
    }

    public string GetBackendName() => "Azure OpenAI (Remote)";
}
```

---

### Option 4: Hybrid Fallback (Local Primary, Remote Fallback)

```csharp
public class HybridEmbeddingService : IEmbeddingService
{
    private readonly IEmbeddingService _primary;
    private readonly IEmbeddingService _fallback;

    public HybridEmbeddingService(IEmbeddingService primary, IEmbeddingService fallback)
    {
        _primary = primary;
        _fallback = fallback;
    }

    public async Task<float[]> EmbedTextAsync(string text)
    {
        try
        {
            return await _primary.EmbedTextAsync(text);
        }
        catch
        {
            Console.WriteLine($"Primary failed ({_primary.GetBackendName()}), using fallback ({_fallback.GetBackendName()})");
            return await _fallback.EmbedTextAsync(text);
        }
    }

    public async Task<List<float[]>> EmbedBatchAsync(List<string> texts)
    {
        try
        {
            return await _primary.EmbedBatchAsync(texts);
        }
        catch
        {
            Console.WriteLine("Primary batch embedding failed, using fallback.");
            return await _fallback.EmbedBatchAsync(texts);
        }
    }

    public string GetBackendName() =>
        $"{_primary.GetBackendName()} (with {_fallback.GetBackendName()} fallback)";
}
```

**Use case:** Development: use local (fast, free). Production: local works 99% of time, Azure handles edge cases or high volume.

---

## 5. Embedding Model Selection: Decision Framework

### Decision Tree: From Constraints to Model

**Step 1: Can we run local SLMs?**
- Yes (on-premise, privacy critical) → Use Ollama or ONNX locally
- No (strict latency <5ms) → Use API-based embeddings
- Maybe → Use hybrid approach

**Step 2: If local - which SLM?**
- Speed critical → ONNX Runtime (20–50ms)
- Development / flexibility → Ollama (100ms, easy model switching)
- Privacy + performance → ONNX in production

**Step 3: If API - which service?**
- Cost-sensitive → OpenAI small ($0.02/1M)
- On-premise requirement → Local only, no API
- Best accuracy → OpenAI large ($0.13/1M)

### Local SLM Embedding Models

| Model | MTEB Rank | Dims | Speed | Memory | Best For |
|-------|-----------|------|-------|--------|----------|
| nomic-embed-text | Top 1 | 768 | ~100ms | 200MB | Ollama, General |
| bge-small-en | Top 5 | 384 | ~50ms | 100MB | ONNX, Speed |
| e5-small | Top 10 | 384 | ~60ms | 120MB | ONNX, Fast retrieval |
| all-MiniLM-L6-v2 | Top 20 | 384 | ~40ms | 80MB | ONNX, Lightweight |

**Recommendation:**
- **Development:** Ollama + nomic-embed-text
- **Production:** ONNX + bge-small-en
- **Maximum performance:** ONNX + nomic-embed-text

---

## 6. Building an Evaluation Framework in .NET

Here's the hard truth: if we're not measuring retrieval quality, we don't know if it is working.

### Step 1: Create Our Test Dataset

```csharp
public class RagTestCase
{
    public string Query { get; set; }
    public string ExpectedAnswer { get; set; }
    public List<string> RelevantChunkIds { get; set; }
    public string Category { get; set; }
    public int DifficultyScore { get; set; }
    public DateTime CreatedDate { get; set; }
    public string Notes { get; set; }
}
```

Build 30–50 test cases covering:
- Different document types
- Different query styles (direct lookups, comparisons, multi-step)
- Edge cases
- Difficulty gradient

**Example:**
```
Query: "What's the refund policy for digital goods?"
Expected: "Refunds allowed within 14 days if unused"
Relevant Chunks: ["doc_5_section_2.3", "doc_5_section_2.4"]
Category: "compliance"
Difficulty: 2
Notes: "Should distinguish from physical goods policy"
```

### Step 2: Define Metrics

```csharp
public class RagEvaluator
{
    public decimal Precision(List<string> retrievedIds, List<string> relevantIds, int k)
    {
        var topK = retrievedIds.Take(k).ToList();
        var hits = topK.Intersect(relevantIds).Count();
        return (decimal)hits / k;
    }

    public decimal Recall(List<string> retrievedIds, List<string> relevantIds, int k)
    {
        var topK = retrievedIds.Take(k).ToList();
        var hits = topK.Intersect(relevantIds).Count();
        return relevantIds.Count == 0 ? 1m : (decimal)hits / relevantIds.Count;
    }

    public int FirstRelevantRank(List<string> retrievedIds, List<string> relevantIds)
    {
        for (int i = 0; i < retrievedIds.Count; i++)
        {
            if (relevantIds.Contains(retrievedIds[i]))
                return i + 1;
        }
        return retrievedIds.Count + 1;
    }

    public Dictionary<string, decimal> AccuracyByCategory(
        List<RagTestCase> testCases,
        Dictionary<string, List<string>> results)
    {
        return testCases
            .GroupBy(t => t.Category)
            .ToDictionary(
                g => g.Key,
                g => g.Average(t =>
                {
                    var retrieved = results[t.Query];
                    return Recall(retrieved, t.RelevantChunkIds, 5);
                })
            );
    }
}
```

### Step 3: Evaluation Loop

```csharp
var baseline = evaluator.EvaluateSystem(testSet);
Console.WriteLine($"Baseline P@5: {baseline.Precision:P}, R@5: {baseline.Recall:P}");

var byCategory = evaluator.AccuracyByCategory(testSet, results);
foreach (var (category, accuracy) in byCategory)
{
    Console.WriteLine($"{category}: {accuracy:P}");
}

// Adjust strategy, re-evaluate
var v2 = evaluator.EvaluateSystem(testSet);
```

**Target:** Aim for P@5 > 70% and R@5 > 65% before going to production.

---

## 7. Complete Extensible End-to-End Example  
### Local SLM, Remote LLM, and Hybrid – One Codebase

### 7.1 Extensible Embedding Service Interface

```csharp
public interface IEmbeddingService
{
    Task<float[]> EmbedTextAsync(string text);
    Task<List<float[]>> EmbedBatchAsync(List<string> texts);
    string GetBackendName();
}
```

**Local (Ollama):**

```csharp
public class OllamaEmbeddingService : IEmbeddingService
{
    private readonly Kernel _kernel;
    private readonly ITextEmbeddingGenerationService _service;

    public OllamaEmbeddingService(Uri ollamaEndpoint)
    {
        var builder = Kernel.CreateBuilder();
        builder.AddOllamaTextEmbedding("nomic-embed-text", ollamaEndpoint);

        _kernel = builder.Build();
        _service = _kernel.GetRequiredService<ITextEmbeddingGenerationService>();
    }

    public async Task<float[]> EmbedTextAsync(string text)
    {
        var embedding = await _service.GenerateEmbeddingAsync(text);
        return embedding.ToArray();
    }

    public async Task<List<float[]>> EmbedBatchAsync(List<string> texts)
    {
        var list = new List<float[]>();
        foreach (var text in texts)
            list.Add(await EmbedTextAsync(text));
        return list;
    }

    public string GetBackendName() => "Ollama (Local)";
}
```

**Remote (Azure OpenAI):**

```csharp
public class AzureOpenAIEmbeddingService : IEmbeddingService
{
    private readonly Kernel _kernel;
    private readonly ITextEmbeddingGenerationService _service;

    public AzureOpenAIEmbeddingService(string deploymentName, Uri endpoint, string apiKey)
    {
        var builder = Kernel.CreateBuilder();
        builder.AddAzureOpenAITextEmbedding(deploymentName, endpoint, apiKey);

        _kernel = builder.Build();
        _service = _kernel.GetRequiredService<ITextEmbeddingGenerationService>();
    }

    public async Task<float[]> EmbedTextAsync(string text)
    {
        var embedding = await _service.GenerateEmbeddingAsync(text);
        return embedding.ToArray();
    }

    public async Task<List<float[]>> EmbedBatchAsync(List<string> texts)
    {
        var list = new List<float[]>();
        foreach (var text in texts)
            list.Add(await EmbedTextAsync(text));
        return list;
    }

    public string GetBackendName() => "Azure OpenAI (Remote)";
}
```

**Hybrid (Local primary, Remote fallback):**

```csharp
public class HybridEmbeddingService : IEmbeddingService
{
    private readonly IEmbeddingService _primary;
    private readonly IEmbeddingService _fallback;

    public HybridEmbeddingService(IEmbeddingService primary, IEmbeddingService fallback)
    {
        _primary = primary;
        _fallback = fallback;
    }

    public async Task<float[]> EmbedTextAsync(string text)
    {
        try
        {
            return await _primary.EmbedTextAsync(text);
        }
        catch
        {
            Console.WriteLine($"Primary failed ({_primary.GetBackendName()}), using fallback ({_fallback.GetBackendName()})");
            return await _fallback.EmbedTextAsync(text);
        }
    }

    public async Task<List<float[]>> EmbedBatchAsync(List<string> texts)
    {
        try
        {
            return await _primary.EmbedBatchAsync(texts);
        }
        catch
        {
            Console.WriteLine("Primary batch embedding failed, using fallback.");
            return await _fallback.EmbedBatchAsync(texts);
        }
    }

    public string GetBackendName() =>
        $"{_primary.GetBackendName()} (with {_fallback.GetBackendName()} fallback)";
}
```

---

### 7.2 Extensible LLM Service Interface

```csharp
public interface ILlmService
{
    Task<string> GenerateAsync(string prompt, string? systemPrompt = null);
    Task<string> GenerateWithContextAsync(string query, string context);
    string GetBackendName();
}
```

**Local (Ollama / Mistral):**

```csharp
public class OllamaLlmService : ILlmService
{
    private readonly Kernel _kernel;
    private readonly IChatCompletionService _service;

    public OllamaLlmService(Uri endpoint)
    {
        var builder = Kernel.CreateBuilder();
        builder.AddOllamaChatCompletion("mistral", endpoint);

        _kernel = builder.Build();
        _service = _kernel.GetRequiredService<IChatCompletionService>();
    }

    public async Task<string> GenerateAsync(string prompt, string? systemPrompt = null)
    {
        var messages = new List<ChatMessageContent>();
        if (!string.IsNullOrEmpty(systemPrompt))
            messages.Add(new ChatMessageContent(AuthorRole.System, systemPrompt));
        messages.Add(new ChatMessageContent(AuthorRole.User, prompt));

        var response = await _service.GetChatMessageContentAsync(messages);
        return response.Content ?? "";
    }

    public Task<string> GenerateWithContextAsync(string query, string context)
    {
        var prompt = @$"
            Based on the following context, answer the user's question concisely.

            CONTEXT:
            {context}

            QUESTION:
            {query}

            ANSWER:";
        return GenerateAsync(prompt);
    }

    public string GetBackendName() => "Ollama (Local Mistral)";
}
```

**Remote (Azure OpenAI GPT‑4):**

```csharp
public class AzureOpenAILlmService : ILlmService
{
    private readonly Kernel _kernel;
    private readonly IChatCompletionService _service;

    public AzureOpenAILlmService(string deploymentName, Uri endpoint, string apiKey)
    {
        var builder = Kernel.CreateBuilder();
        builder.AddAzureOpenAIChatCompletion(deploymentName, endpoint, apiKey);

        _kernel = builder.Build();
        _service = _kernel.GetRequiredService<IChatCompletionService>();
    }

    public async Task<string> GenerateAsync(string prompt, string? systemPrompt = null)
    {
        var messages = new List<ChatMessageContent>();
        if (!string.IsNullOrEmpty(systemPrompt))
            messages.Add(new ChatMessageContent(AuthorRole.System, systemPrompt));
        messages.Add(new ChatMessageContent(AuthorRole.User, prompt));

        var response = await _service.GetChatMessageContentAsync(messages);
        return response.Content ?? "";
    }

    public Task<string> GenerateWithContextAsync(string query, string context)
    {
        var prompt = @$"
            Based on the following context, answer the user's question concisely.

            CONTEXT:
            {context}

            QUESTION:
            {query}

            ANSWER:";
        return GenerateAsync(prompt);
    }

    public string GetBackendName() => "Azure OpenAI (Remote GPT‑4)";
}
```

---

### 7.3 Vector Store and Metrics

```csharp
public class ChunkWithEmbedding
{
    public string Id { get; set; }
    public string Text { get; set; }
    public Dictionary<string, string> Metadata { get; set; }
    public float[] Embedding { get; set; }
}

public class VectorStore
{
    private readonly List<ChunkWithEmbedding> _chunks = new();

    public void AddChunk(ChunkWithEmbedding chunk) => _chunks.Add(chunk);

    public List<ChunkWithEmbedding> SearchSimilar(float[] queryEmbedding, int k)
    {
        return _chunks
            .Select(c => (Chunk: c, Similarity: CosineSimilarity(queryEmbedding, c.Embedding)))
            .OrderByDescending(x => x.Similarity)
            .Take(k)
            .Select(x => x.Chunk)
            .ToList();
    }

    private static float CosineSimilarity(float[] a, float[] b)
    {
        float dot = 0, normA = 0, normB = 0;
        for (int i = 0; i < a.Length; i++)
        {
            dot += a[i] * b[i];
            normA += a[i] * a[i];
            normB += b[i] * b[i];
        }
        return dot / (float)(Math.Sqrt(normA) * Math.Sqrt(normB));
    }
}

public class QueryMetrics
{
    public string Query { get; set; }
    public int RetrievedChunks { get; set; }
    public int RetrievalLatencyMs { get; set; }
    public int GenerationLatencyMs { get; set; }
    public int TotalLatencyMs { get; set; }
    public DateTime Timestamp { get; set; }
    public string EmbeddingBackend { get; set; }
    public string LlmBackend { get; set; }
}
```

---

### 7.4 ExtensibleDocumentRAGSystem

```csharp
public class ExtensibleDocumentRAGSystem
{
    private readonly IEmbeddingService _embeddingService;
    private readonly ILlmService _llmService;
    private readonly VectorStore _vectorStore = new();
    private readonly List<QueryMetrics> _metrics = new();

    public ExtensibleDocumentRAGSystem(IEmbeddingService embeddingService, ILlmService llmService)
    {
        _embeddingService = embeddingService;
        _llmService = llmService;

        Console.WriteLine($"✓ Embedding backend: {embeddingService.GetBackendName()}");
        Console.WriteLine($"✓ LLM backend: {llmService.GetBackendName()}");
    }

    public async Task IngestDocumentsAsync(List<string> docs)
    {
        var chunker = new DocumentationChunker();
        var totalChunks = 0;

        foreach (var doc in docs)
        {
            var chunks = chunker.ChunkMarkdownDocument(doc);
            foreach (var chunk in chunks)
            {
                var embedding = await _embeddingService.EmbedTextAsync(chunk.Text);
                _vectorStore.AddChunk(new ChunkWithEmbedding
                {
                    Id = Guid.NewGuid().ToString(),
                    Text = chunk.Text,
                    Metadata = chunk.Metadata,
                    Embedding = embedding
                });
                totalChunks++;
            }
        }

        Console.WriteLine($"✓ Ingested {docs.Count} documents, {totalChunks} chunks total");
    }

    private async Task<List<ChunkWithEmbedding>> RetrieveAsync(string query, int k)
    {
        var queryEmbedding = await _embeddingService.EmbedTextAsync(query);
        return _vectorStore.SearchSimilar(queryEmbedding, k);
    }

    public async Task<(string answer, QueryMetrics metrics)> GenerateAnswerAsync(string query)
    {
        var sw = System.Diagnostics.Stopwatch.StartNew();
        var start = DateTime.UtcNow;

        var retrievalStart = sw.ElapsedMilliseconds;
        var chunks = await RetrieveAsync(query, 5);
        var retrievalMs = (int)(sw.ElapsedMilliseconds - retrievalStart);

        if (chunks.Count == 0)
        {
            return ("No relevant information found.", new QueryMetrics
            {
                Query = query,
                RetrievedChunks = 0,
                RetrievalLatencyMs = retrievalMs,
                GenerationLatencyMs = 0,
                TotalLatencyMs = (int)sw.ElapsedMilliseconds,
                Timestamp = start,
                EmbeddingBackend = _embeddingService.GetBackendName(),
                LlmBackend = _llmService.GetBackendName()
            });
        }

        var context = new StringBuilder();
        foreach (var c in chunks)
            context.AppendLine($"- {c.Text.Substring(0, Math.Min(150, c.Text.Length))}...");

        var genStart = sw.ElapsedMilliseconds;
        var answer = await _llmService.GenerateWithContextAsync(query, context.ToString());
        var genMs = (int)(sw.ElapsedMilliseconds - genStart);

        sw.Stop();

        var metrics = new QueryMetrics
        {
            Query = query,
            RetrievedChunks = chunks.Count,
            RetrievalLatencyMs = retrievalMs,
            GenerationLatencyMs = genMs,
            TotalLatencyMs = (int)sw.ElapsedMilliseconds,
            Timestamp = start,
            EmbeddingBackend = _embeddingService.GetBackendName(),
            LlmBackend = _llmService.GetBackendName()
        };

        _metrics.Add(metrics);

        return (answer, metrics);
    }

    public void PrintMetricsSummary()
    {
        if (_metrics.Count == 0)
        {
            Console.WriteLine("No queries yet.");
            return;
        }

        Console.WriteLine("\n==============================");
        Console.WriteLine("QUERY METRICS SUMMARY");
        Console.WriteLine("==============================");

        var avgRetrieval = _metrics.Average(m => m.RetrievalLatencyMs);
        var avgGen = _metrics.Average(m => m.GenerationLatencyMs);
        var avgTotal = _metrics.Average(m => m.TotalLatencyMs);

        Console.WriteLine($"Total queries: {_metrics.Count}");
        Console.WriteLine($"Embedding backend: {_embeddingService.GetBackendName()}");
        Console.WriteLine($"LLM backend: {_llmService.GetBackendName()}");
        Console.WriteLine($"Retrieval latency (avg): {avgRetrieval:F0} ms");
        Console.WriteLine($"Generation latency (avg): {avgGen:F0} ms");
        Console.WriteLine($"Total latency (avg): {avgTotal:F0} ms\n");

        foreach (var m in _metrics)
        {
            Console.WriteLine($"Q: \"{m.Query}\"");
            Console.WriteLine($"   Retrieval: {m.RetrievalLatencyMs} ms | Generation: {m.GenerationLatencyMs} ms | Total: {m.TotalLatencyMs} ms");
        }

        Console.WriteLine("==============================");
    }
}
```

---

### 7.5 Usage Scenarios

```csharp
public class Program
{
    public static async Task Main()
    {
        Console.WriteLine("=== EXTENSIBLE RAG SYSTEM DEMO ===\n");

        Console.WriteLine("SCENARIO 1: Local SLMs (Ollama)\n");
        await RunLocalOnlyDemo();

        Console.WriteLine("\n----------------------------------------\n");

        Console.WriteLine("SCENARIO 2: Hybrid (Local with Azure fallback)\n");
        await RunHybridDemo();

        Console.WriteLine("\n----------------------------------------\n");

        Console.WriteLine("SCENARIO 3: Azure OpenAI only\n");
        await RunAzureOnlyDemo();
    }

    private static async Task RunLocalOnlyDemo()
    {
        var embedding = new OllamaEmbeddingService(new Uri("http://localhost:11434"));
        var llm = new OllamaLlmService(new Uri("http://localhost:11434"));
        var rag = new ExtensibleDocumentRAGSystem(embedding, llm);

        var docs = new List<string>
        {
            @"# Authentication Guide
                ## OAuth 2.0
                Our API uses OAuth 2.0 for authentication.
                Steps: Register app → Get credentials → Exchange for token",

            @"# Refund Policy
                ## Digital Goods
                Digital goods can be refunded within 14 days if unused."
        };

        await rag.IngestDocumentsAsync(docs);

        var (answer, metrics) = await rag.GenerateAnswerAsync("How do I authenticate?");
        Console.WriteLine($"Q: How do I authenticate?");
        Console.WriteLine($"A: {answer}");
        Console.WriteLine($"Latency: {metrics.TotalLatencyMs} ms\n");

        rag.PrintMetricsSummary();
    }

    private static async Task RunHybridDemo()
    {
        var localEmbedding = new OllamaEmbeddingService(new Uri("http://localhost:11434"));
        var azureEmbedding = new AzureOpenAIEmbeddingService(
            "text-embedding-3-small",
            new Uri("https://[resource].openai.azure.com/"),
            "Our-key");

        var hybrid = new HybridEmbeddingService(localEmbedding, azureEmbedding);
        var llm = new OllamaLlmService(new Uri("http://localhost:11434"));

        var rag = new ExtensibleDocumentRAGSystem(hybrid, llm);
        Console.WriteLine("Hybrid embedding: local primary, Azure fallback.");
    }

    private static async Task RunAzureOnlyDemo()
    {
        var embedding = new AzureOpenAIEmbeddingService(
            "text-embedding-3-small",
            new Uri("https://[resource].openai.azure.com/"),
            "Our-key");

        var llm = new AzureOpenAILlmService(
            "gpt-4",
            new Uri("https://[resource].openai.azure.com/"),
            "Our-key");

        var rag = new ExtensibleDocumentRAGSystem(embedding, llm);
        Console.WriteLine("Using Azure OpenAI for embeddings and LLM.");
    }
}
```

With this design, moving from local-only to cloud-only or hybrid is a constructor change, not a rewrite.

---

## 8. Deployment Checklist

Before deploying:

**Local SLM Setup:**
- Ollama or ONNX runtime installed and tested
- Embedding model downloaded and verified
- LLM model (Mistral/Llama) tested locally
- Performance benchmarked (latency, memory usage)

**Vector Store:**
- Qdrant or Weaviate deployed (separate from app)
- Backup/recovery strategy in place
- Index size estimated and storage allocated

**Monitoring:**
- Latency tracked (retrieval + generation)
- Cost tracked (if using any API calls)
- Error rates monitored
- Chunk quality metrics tracked

**Evaluation:**
- Test set created (30–50 queries)
- P@5 > 70%, R@5 > 65% achieved
- Edge cases tested
- Performance regression detection in place

---

## 9. Deployment Scenarios at a Glance

| Scenario | Embedding | LLM | Cost | Latency | Privacy |
|----------|-----------|-----|------|---------|---------|
| **Development** | Ollama | Ollama | 0 USD | ~500ms | ✅ Local |
| **On-Premise** | ONNX | Ollama | 0 USD | ~200ms | ✅ Local |
| **Cloud** | Azure | Azure | 20–100 USD/mo | ~100ms | ⚠️ Azure |
| **Hybrid (Safe)** | Ollama→Azure | Ollama | 5–50 USD/mo | ~150ms | ✅ Local primary |

---

## 10. Where This Fits: RAG as Our Agent's Knowledge Layer

This RAG pipeline with pluggable backends becomes the **knowledge layer** for agents:

- **Single-turn Q&A**: Retrieve docs, generate answer
- **Multi-turn conversations**: Maintain context, retrieve relevant updates
- **Tool-using agents**: RAG as a tool/function the agent can call
- **Workflow automation**: Agents orchestrate retrieval + decision-making

Semantic Kernel plugins let we create sophisticated pipelines entirely in .NET.

---

## Conclusion

we now have:
- **Five architecture patterns** to pick from based on constraints
- **Domain-specific chunking** for technical, legal, code, FAQ, and time-series data
- **Local and remote embedding options**, all behind a single interface
- **Evaluation framework** to measure and improve retrieval accuracy
- **Observable metrics** for every layer of the pipeline

RAG is infrastructure. Build it intentionally, measure it rigorously, and keep it simple until we have evidence we need complexity.

Start with Pattern 1 (Simple Sequential) + local SLMs (Ollama). Measure retrieval accuracy. If we need better, add hybrid retrieval. If we need speed, move to ONNX. If we need maximum accuracy, switch to Azure—without rewriting business logic.

That is reliable RAG.

**Reference:** [Semantic Kernel](https://learn.microsoft.com/en-us/semantic-kernel/).

