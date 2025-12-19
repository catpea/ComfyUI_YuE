### Short answer

It asks for a **model path / repo string** because the node is written to load models via
`AutoModelForCausalLM.from_pretrained()` instead of ComfyUI’s model selector system.
So ComfyUI has **nothing to list** in a dropdown.

Yes — **this can be fixed**, and cleanly.

---

## Why this happens (in your code)

### 1. ComfyUI model dropdowns only work if:

* The node uses `folder_paths.get_filename_list("model_type")`
* And loads models via `folder_paths.get_full_path(...)`

Example of a *proper* ComfyUI selector:

```python
ckpt_list = folder_paths.get_filename_list("checkpoints")
```

Your YUE nodes **do this correctly** for:

* xcodec checkpoints
* vocal decoders
* instrumental decoders

### 2. But Stage A / Stage B models are loaded like this:

```python
AutoModelForCausalLM.from_pretrained(stage_A_repo)
```

Where `stage_A_repo` is a **STRING**, e.g.:

```text
m-a-p/YuE-s1-7B-anneal-en-cot
```

That means:

* Hugging Face downloads it automatically
* ComfyUI never sees it
* No dropdown is possible

So ComfyUI *must* ask you for a string/path.

---

## The correct ComfyUI-style fix

### Goal

Make Stage A & Stage B models:

* Appear in a dropdown
* Live in `ComfyUI/models/yue/`
* Work **offline**
* Behave like every other ComfyUI node

---

## Step 1 — Add a model category

Near the top (you already do this for yue weights):

```python
YUE_LLM_PATH = os.path.join(folder_paths.models_dir, "yue_llm")
os.makedirs(YUE_LLM_PATH, exist_ok=True)
folder_paths.add_model_folder_path("yue_llm", YUE_LLM_PATH)
```

This creates:

```
ComfyUI/models/yue_llm/
```

---

## Step 2 — Change INPUT_TYPES to use a dropdown

### Before (current, broken UX)

```python
"stage_A_repo": ("STRING", {"default": "m-a-p/YuE-s1-7B-anneal-en-cot"}),
```

### After (ComfyUI-style)

```python
@classmethod
def INPUT_TYPES(s):
    llm_list = folder_paths.get_filename_list("yue_llm")
    return {
        "required": {
            "stage_A_model": (llm_list,),
            ...
        }
    }
```

Now ComfyUI will show a dropdown.

---

## Step 3 — Load the model from ComfyUI’s path

### Replace:

```python
stage_1_model = AutoModelForCausalLM.from_pretrained(stage_A_repo)
```

### With:

```python
model_path = folder_paths.get_full_path("yue_llm", stage_A_model)

stage_1_model = AutoModelForCausalLM.from_pretrained(
    model_path,
    torch_dtype=torch.bfloat16,
    attn_implementation="flash_attention_2",
)
```

Same for:

* `Stage1Pipeline_HF`
* `Stage1Pipeline_EXL2`
* Stage B loaders

---

## Step 4 — How models should be stored

You would place models like this:

```
ComfyUI/models/yue_llm/
├── YuE-s1-7B-anneal-en-cot/
│   ├── config.json
│   ├── model.safetensors
│   └── tokenizer.json
└── YuE-s2-1B-general/
    ├── config.json
    └── model.safetensors
```

You can:

* `git clone` from Hugging Face
* or use `huggingface-cli download`
* or copy from `~/.cache/huggingface/hub`

---

## Optional: hybrid mode (best UX)

You can allow **both**:

* Dropdown selection
* Manual HF repo fallback

Example:

```python
if stage_A_model == "hf:YuE-s1-7B":
    model_path = "m-a-p/YuE-s1-7B-anneal-en-cot"
else:
    model_path = folder_paths.get_full_path("yue_llm", stage_A_model)
```

This keeps compatibility without breaking ComfyUI UX.

---

## Bottom line

**Why it happens**

* Node is written like a Hugging Face script, not a ComfyUI node

