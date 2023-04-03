# llama-node

This project is in an early stage, the API for nodejs may change in the future, use it with caution.

<img src="./doc/assets/llama.jpeg" width="300px" height="300px" alt="LLaMA generated by bing creator"/>


![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/hlhr202/llama-node/llama-build.yml)
![NPM](https://img.shields.io/npm/l/llama-node)
[<img alt="npm" src="https://img.shields.io/npm/v/llama-node">](https://www.npmjs.com/package/llama-node)
![npm type definitions](https://img.shields.io/npm/types/llama-node)

# Introduction

This is a nodejs client library for llama LLM built on top of [llama-rs](https://github.com/rustformers/llama-rs). It uses [napi-rs](https://github.com/napi-rs/napi-rs) for channel messages between node.js and llama thread.

Currently supported platforms:
- darwin-x64
- darwin-arm64
- linux-x64-gnu
- win32-x64-msvc


I do not have hardware for testing 13B or larger models, but I have tested it supported llama 7B model with both ggml llama and ggml alpaca.

Download one of the llama ggml models from the following links:
- [llama 7B int4 (old model for llama.cpp)](https://huggingface.co/hlhr202/llama-7B-ggml-int4/blob/main/ggml-model-q4_0.bin)
- [alpaca 7B int4](https://huggingface.co/hlhr202/alpaca-7B-ggml-int4/blob/main/ggml-alpaca-7b-q4.bin)
---

## Install
```bash
npm install llama-node
```

---

## Usage

The current version supports only one inference session on one LLama instance at the same time

If you wish to have multiple inference sessions concurrently, you need to create multiple LLama instances

### Inference

```typescript
import path from "path";
import { LLamaClient } from "llama-node";

const model = path.resolve(process.cwd(), "./ggml-alpaca-7b-q4.bin");

const llama = new LLamaClient(
    {
        path: model,
        numCtxTokens: 128,
    },
    true
);

const template = `how are you`;

const prompt = `Below is an instruction that describes a task. Write a response that appropriately completes the request.

### Instruction:

${template}

### Response:`;

llama.createTextCompletion(
    {
        prompt,
        numPredict: 128,
        temp: 0.2,
        topP: 1,
        topK: 40,
        repeatPenalty: 1,
        repeatLastN: 64,
        seed: 0,
        feedPrompt: true,
    },
    (response) => {
        process.stdout.write(response.token);
    }
);
```

### Chatting

Working on alpaca, this just make a context of alpaca instructions. Make sure your last message is end with user role.

```typescript
import { LLamaClient } from "llama-node";
import path from "path";

const model = path.resolve(process.cwd(), "./ggml-alpaca-7b-q4.bin");

const llama = new LLamaClient(
    {
        path: model,
        numCtxTokens: 128,
    },
    true
);

const content = "how are you?";

llama.createChatCompletion(
    {
        messages: [{ role: "user", content }],
        numPredict: 128,
        temp: 0.2,
        topP: 1,
        topK: 40,
        repeatPenalty: 1,
        repeatLastN: 64,
        seed: 0,
    },
    (response) => {
        if (!response.completed) {
            process.stdout.write(response.token);
        }
    }
);

```

### Tokenize

Get tokenization result from LLaMA

```typescript
import { LLamaClient } from "llama-node";
import path from "path";

const model = path.resolve(process.cwd(), "./ggml-alpaca-7b-q4.bin");

const llama = new LLamaClient(
    {
        path: model,
        numCtxTokens: 128,
    },
    true
);

const content = "how are you?";

llama.tokenize(content).then(console.log);
```

### Embedding

Preview version, embedding end token may change in the future. Do not use it in production!

```typescript
import { LLamaClient } from "llama-node";
import path from "path";

const model = path.resolve(process.cwd(), "./ggml-alpaca-7b-q4.bin");

const llama = new LLamaClient(
    {
        path: model,
        numCtxTokens: 128,
    },
    true
);

const prompt = `how are you`;

llama
    .getEmbedding({
        prompt,
        numPredict: 128,
        temp: 0.2,
        topP: 1,
        topK: 40,
        repeatPenalty: 1,
        repeatLastN: 64,
        seed: 0,
        feedPrompt: true,
    })
    .then(console.log);

```

---

## Self built

Make sure you have installed rust

```bash
cd packages/core
npm run build
```

---

## Future plan
- [ ] prompt extensions
- [ ] more platforms and cross compile
- [ ] tweak embedding API, make end token configurable
