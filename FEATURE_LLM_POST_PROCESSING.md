# LLM Post-Processing Feature

**Branch:** `feature/llm-post-processing`  
**Base:** `main`

## Overview

This feature adds an additional window that processes speech-to-text output through a llama.cpp model with custom prompts, allowing users to transform their transcribed text (e.g., make it more professional, change tone, fix grammar, etc.).

## Architecture

### Workflow
1. User speaks → Handy captures audio
2. Whisper transcribes speech → raw STT text
3. **[NEW]** Raw text + user prompt → llama.cpp model → processed text
4. Processed text → clipboard/paste to active application

### Components to Add

#### Backend (Rust - src-tauri/src/)

**New Files:**
- `managers/llm.rs` - LLM model management and inference
  - Model downloading/loading (llama.cpp models)
  - Prompt template management
  - Text processing pipeline
  
- `commands/llm.rs` - Tauri commands for LLM operations
  - `download_llm_model(model_id: String)`
  - `load_llm_model(model_path: String)`
  - `process_text(text: String, prompt: String)`
  - `get_available_prompts()`
  - `save_custom_prompt(name: String, template: String)`

**Dependencies to Add (Cargo.toml):**
```toml
llama-cpp-rs = "0.1" # or appropriate llama.cpp Rust bindings
```

#### Frontend (React/TypeScript - src/)

**New Files:**
- `components/llm-window/LlmWindow.tsx` - Main LLM processing window
- `components/llm-window/ModelSelector.tsx` - Select llama.cpp model
- `components/llm-window/PromptEditor.tsx` - Create/edit prompts
- `components/llm-window/PromptPresets.tsx` - Preset prompt templates
- `hooks/useLlmProcessing.ts` - LLM processing logic hook

**Settings to Add:**
- Enable/disable LLM post-processing
- Selected LLM model
- Active prompt template
- Custom prompts library
- Processing mode (always/shortcut/manual)

### Prompt Templates (Examples)

```
Professional: "Rewrite the following text to be more professional and formal: {text}"
Concise: "Summarize the following text concisely: {text}"
Grammar Fix: "Fix grammar and spelling in: {text}"
Casual: "Make this text more casual and friendly: {text}"
Technical: "Rewrite this in technical documentation style: {text}"
```

## Integration Points

### Modified Files
- `src-tauri/src/lib.rs` - Register LLM manager and commands
- `src-tauri/src/managers/transcription.rs` - Add optional LLM processing step
- `src-tauri/src/settings.rs` - Add LLM settings
- `src/App.tsx` - Add LLM window management
- `src/components/settings/` - Add LLM settings panel
- `src-tauri/resources/default_settings.json` - Add LLM defaults

## Development Strategy

### Phase 1: Basic Infrastructure
- [ ] Add llama.cpp Rust bindings dependency
- [ ] Create LLM manager for model loading/inference
- [ ] Add basic Tauri commands
- [ ] Create LLM settings structure

### Phase 2: UI Components
- [ ] Build LLM window component
- [ ] Add model selector UI
- [ ] Create prompt editor
- [ ] Add preset prompt templates

### Phase 3: Integration
- [ ] Integrate with transcription pipeline
- [ ] Add settings panel
- [ ] Implement enable/disable toggle
- [ ] Add keyboard shortcut for LLM window

### Phase 4: Polish
- [ ] Add model download progress
- [ ] Error handling and user feedback
- [ ] Performance optimization
- [ ] Documentation

## Maintaining Fork

### Workflow for Upstream Updates

```bash
# Add upstream remote (original handy repo)
git remote add upstream https://github.com/HandyHQ/handy.git

# Fetch upstream changes
git fetch upstream

# Update local main branch
git checkout main
git merge upstream/main
git push origin main

# Rebase feature branch on updated main
git checkout feature/llm-post-processing
git rebase main

# Or merge if preferred
git merge main
```

### Handling Conflicts
- Keep LLM-specific changes separate
- Document any modifications to existing files
- Use feature flags where possible
- Maintain backward compatibility

## Model Recommendations

**Small & Fast:**
- Llama 3.2 1B/3B
- Phi-3 Mini

**Balanced:**
- Llama 3.2 8B
- Mistral 7B

**High Quality:**
- Llama 3.1 8B
- Qwen 2.5 7B

## Configuration Example

```json
{
  "llm": {
    "enabled": false,
    "model_path": "",
    "active_prompt": "professional",
    "custom_prompts": {
      "professional": "Rewrite the following text to be more professional: {text}",
      "concise": "Summarize concisely: {text}"
    },
    "processing_mode": "always", // "always" | "shortcut" | "manual"
    "show_original": true,
    "auto_apply": false
  }
}
```

## Notes

- LLM processing is optional and can be toggled on/off
- Consider memory usage with model loading
- May want to unload LLM model when not in use (similar to Whisper)
- Window should show both original STT and processed text
- User can choose to use original or processed version
