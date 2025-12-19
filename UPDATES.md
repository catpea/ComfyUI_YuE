# ComfyUI_YuE Updates

## Latest Changes

### Fixed Issues from Bug Report

1. **Added ComfyUI-style Model Selector for Stage A/B Models**
   - Models now appear in dropdown menus instead of requiring manual text input
   - Proper offline support with local model storage
   - Follows ComfyUI conventions for model management

2. **New Model Directory Structure**
   - Created new model folder: `ComfyUI/models/yue_llm/`
   - Stage A and Stage B LLM models should be placed here
   - Models are now managed through ComfyUI's model system

### How to Use the Updated Version

#### Option 1: Local Model Storage (Recommended)

Place your downloaded Stage A and Stage B models in the `ComfyUI/models/yue_llm/` directory:

```
ComfyUI/models/yue_llm/
├── YuE-s1-7B-anneal-en-cot/
│   ├── config.json
│   ├── model.safetensors.index.json
│   ├── tokenizer.model
│   ├── model-00001-of-00003.safetensors
│   ├── model-00002-of-00003.safetensors
│   └── model-00003-of-00003.safetensors
└── YuE-s2-1B-general/
    ├── config.json
    ├── model.safetensors
    └── tokenizer.model
```

**To download models manually:**

```bash
# Using huggingface-cli
huggingface-cli download m-a-p/YuE-s1-7B-anneal-en-cot --local-dir ComfyUI/models/yue_llm/YuE-s1-7B-anneal-en-cot
huggingface-cli download m-a-p/YuE-s2-1B-general --local-dir ComfyUI/models/yue_llm/YuE-s2-1B-general

# Or using git
cd ComfyUI/models/yue_llm/
git clone https://huggingface.co/m-a-p/YuE-s1-7B-anneal-en-cot
git clone https://huggingface.co/m-a-p/YuE-s2-1B-general

# Or copy from HuggingFace cache
cp -r ~/.cache/huggingface/hub/models--m-a-p--YuE-s1-7B-anneal-en-cot/snapshots/[hash]/* ComfyUI/models/yue_llm/YuE-s1-7B-anneal-en-cot/
```

#### Option 2: HuggingFace Auto-Download (Fallback)

If you don't have local models, you can still use the dropdown to select HuggingFace repos:
- Select options prefixed with `hf:` (e.g., `hf:m-a-p/YuE-s1-7B-anneal-en-cot`)
- Models will be automatically downloaded from HuggingFace on first use
- Downloaded models are cached by HuggingFace in `~/.cache/huggingface/`

### Benefits of These Changes

1. **Better UX**: No more typing repo strings manually
2. **Offline Support**: Works without internet once models are downloaded
3. **ComfyUI Standard**: Follows the same pattern as other ComfyUI nodes
4. **Flexible**: Supports both local models and HuggingFace auto-download
5. **Easy Model Management**: See all available models in dropdown

### Available Stage A Models

- **English (CoT)**: YuE-s1-7B-anneal-en-cot
- **English (ICL)**: YuE-s1-7B-anneal-en-icl
- **Chinese (CoT)**: YuE-s1-7B-anneal-zh-cot
- Quantized versions (exllamav2, int8, int4) also supported

### Available Stage B Models

- **General**: YuE-s2-1B-general
- Quantized versions also supported

### Backward Compatibility

The updated code maintains compatibility with:
- All quantization methods (fp16, int8, int4, exllamav2)
- MMGP memory optimization
- All existing features and settings

### What Changed in the Code

1. Added `yue_llm` model folder registration (yue_node.py:38-42)
2. Updated `YUE_Stage_A_Loader.INPUT_TYPES` to use dropdown (yue_node.py:49-70)
3. Updated `YUE_Stage_A_Loader.loader_main` to support hybrid loading (yue_node.py:77-90)
4. Updated `YUE_Stage_B_Loader.INPUT_TYPES` to use dropdown (yue_node.py:419-436)
5. Updated `YUE_Stage_B_Loader.loader_main` to support hybrid loading (yue_node.py:443-456)

### Migration Guide

If you were using the old version with HuggingFace repo strings:

**Before:**
- `stage_A_repo`: "m-a-p/YuE-s1-7B-anneal-en-cot" (typed manually)

**After:**
- `stage_A_model`: Select from dropdown
  - Choose local model if available (e.g., "YuE-s1-7B-anneal-en-cot")
  - OR choose HuggingFace option (e.g., "hf:m-a-p/YuE-s1-7B-anneal-en-cot")

No other changes to your workflow are required.
