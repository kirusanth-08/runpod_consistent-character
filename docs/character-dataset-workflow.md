# Character Dataset Workflow Guide

## Overview

The Character Dataset workflow enables consistent character generation with image-to-image editing capabilities using Flux 2 Klein model. This workflow allows you to transform input character images into new scenes while maintaining character consistency.

## Features

- **Consistent Character Generation**: Maintains character appearance across different scenes
- **Image-to-Image Editing**: Transform character images with natural language prompts
- **High Resolution Output**: Supports up to 3072x3072 output resolution
- **Fast Generation**: Uses Flux 2 Klein model with optimized 4-step sampling
- **LoRA Enhancement**: Includes NSFW v2 LoRA for enhanced quality

## Model Requirements

The workflow requires the following models to be available in your ComfyUI installation:

### UNET Model
- **File**: `flux-2-klein-9b.safetensors`
- **Location**: `ComfyUI/models/unet/`
- **Description**: Flux 2 Klein 9B diffusion model

### CLIP Model
- **File**: `qwen_3_8b_fp8mixed.safetensors`
- **Location**: `ComfyUI/models/clip/`
- **Type**: flux2
- **Description**: Qwen 3.8B CLIP model for text encoding

### VAE Model
- **File**: `flux2-vae.safetensors`
- **Location**: `ComfyUI/models/vae/`
- **Description**: Flux 2 VAE for encoding/decoding

### LoRA Model
- **File**: `Flux Klein - NSFW v2.safetensors`
- **Location**: `ComfyUI/models/loras/`
- **Strength**: 0.2
- **Description**: Enhancement LoRA for improved quality

## Workflow Components

The workflow consists of the following key nodes:

1. **Load Image (Node 125)**: Loads the input character image
2. **Image Scale (Node 104)**: Scales input to 1 megapixel for processing
3. **VAE Encode (Node 105)**: Encodes the input image to latent space
4. **CLIP Text Encode (Node 119)**: Processes the text prompt
5. **Reference Latent (Nodes 107, 108)**: Creates positive and negative conditioning
6. **Sampler (Node 99)**: Advanced sampling with 4 steps
7. **VAE Decode (Node 106)**: Decodes output to image
8. **Save Image (Node 121)**: Saves the final output

## Usage Example

### Minimal Request Structure (Recommended)

The API now supports a simplified format that only requires essential parameters:

```json
{
  "input": {
    "prompt": "Make this person on the image standing on a ground between flower plants",
    "image": "data:image/png;base64,<your_base64_encoded_image>",
    "seed": 148059131098564,
    "width": 1024,
    "height": 1024
  }
}
```

**Required Fields:**
- `prompt` (string): Text description of the desired transformation
- `image` (string): Base64 encoded input image (with or without data URI prefix)
- `seed` (integer): Random seed for reproducible results
- `width` (integer): Output image width in pixels
- `height` (integer): Output image height in pixels

**Optional Fields (with defaults):**
- `cfg` (float): CFG scale, default: 1
- `sampler_name` (string): Sampler to use, default: "euler"
- `steps` (integer): Number of sampling steps, default: 4
- `lora_strength` (float): LoRA strength, default: 0.2

### Full Workflow Request Structure

For advanced use cases, you can still provide the complete workflow:

```json
{
  "input": {
    "workflow": {
      // Complete workflow JSON from workflow_character_dataset.json
    },
    "images": [
      {
        "name": "input_character.png",
        "image": "data:image/png;base64,<your_base64_encoded_image>"
      }
    ]
  }
}
```

### Customizing the Prompt

Simply change the `prompt` field in your request:

```json
{
  "input": {
    "prompt": "Your custom prompt here",
    "image": "data:image/png;base64,...",
    "seed": 42,
    "width": 1024,
    "height": 1024
  }
}
```

For full workflow format, modify the `text` field in node 119:

```json
"119": {
  "inputs": {
    "text": "Your custom prompt here",
    "clip": ["100", 0]
  },
  "class_type": "CLIPTextEncode",
  "_meta": {
    "title": "CLIP Text Encode (Positive Prompt)"
  }
}
```

### Example Prompts

- `"Make this person on the image standing on a ground between flower plants"`
- `"Portrait of this character in a cyberpunk city at night"`
- `"This person wearing a spacesuit on Mars"`
- `"Character sitting in a cozy coffee shop reading a book"`

## Advanced Configuration

### Using Minimal Format with Optional Parameters

```json
{
  "input": {
    "prompt": "Portrait of this character in a cyberpunk city",
    "image": "data:image/png;base64,...",
    "seed": 42,
    "width": 2048,
    "height": 2048,
    "cfg": 1.5,
    "steps": 8,
    "sampler_name": "euler",
    "lora_strength": 0.3
  }
}
```

### Adjusting Resolution (Full Workflow)

Modify node 102 to change output resolution:

```json
"102": {
  "inputs": {
    "width": 2048,    // Change width
    "height": 2048,   // Change height
    "batch_size": 1
  },
  "class_type": "EmptyFlux2LatentImage"
}
```

### Adjusting Sampling Steps

Modify node 103 to change quality/speed tradeoff:

```json
"103": {
  "inputs": {
    "steps": 8,  // Increase for better quality (slower)
    "width": ["117", 0],
    "height": ["117", 1]
  },
  "class_type": "Flux2Scheduler"
}
```

### Adjusting LoRA Strength

Modify node 116 to change LoRA influence:

```json
"116": {
  "inputs": {
    "lora_name": "Flux Klein - NSFW v2.safetensors",
    "strength_model": 0.3,  // Range: 0.0 to 1.0
    "model": ["113", 0]
  }
}
```

### Changing Random Seed

Modify node 109 for different variations:

```json
"109": {
  "inputs": {
    "noise_seed": 42  // Change for different results
  },
  "class_type": "RandomNoise"
}
```

## Docker Configuration

If building a custom Docker image with this workflow, ensure the Dockerfile includes:

```dockerfile
# Copy the snapshot file with required custom nodes
COPY snapshot-style_ref.json /snapshot-style_ref.json

# Restore snapshot to install dependencies
COPY src/restore_snapshot.sh /restore_snapshot.sh
RUN chmod +x /restore_snapshot.sh && \
    /restore_snapshot.sh
```

## Custom Nodes Required

The workflow requires these custom nodes (installed via snapshot):

- **ComfyUI-GGUF**: For GGUF model loading support
- **ComfyUI-KJNodes**: Additional utility nodes
- **ComfyUI-Impact-Pack**: Image processing utilities

These are automatically installed when using the snapshot file.

## Troubleshooting

### Model Not Found Errors

If you see errors like "model not found", ensure:

1. All model files are in the correct directories
2. File names match exactly (case-sensitive)
3. Models are downloaded completely (check file sizes)

### Out of Memory Errors

If you encounter OOM errors:

1. Reduce the output resolution in node 102
2. Reduce batch_size to 1
3. Use a GPU with more VRAM (24GB+ recommended)

### Poor Quality Results

To improve output quality:

1. Increase sampling steps in node 103 (e.g., 8 or 12)
2. Adjust LoRA strength in node 116
3. Refine your text prompt to be more specific
4. Try different random seeds

## API Integration Example

### Python Example (Minimal Format)

```python
import requests
import base64

# Read and encode your image
with open("character.png", "rb") as f:
    image_data = base64.b64encode(f.read()).decode("utf-8")

# Prepare minimal request
payload = {
    "input": {
        "prompt": "Make this person standing in a beautiful garden",
        "image": f"data:image/png;base64,{image_data}",
        "seed": 123456789,
        "width": 1024,
        "height": 1024
    }
}

# Make request
response = requests.post(
    f"https://api.runpod.ai/v2/{ENDPOINT_ID}/runsync",
    headers={
        "Authorization": f"Bearer {API_KEY}",
        "Content-Type": "application/json"
    },
    json=payload
)

# Get result
result = response.json()
output_image_base64 = result["output"]["images"][0]["data"]
```

### Python Example (Full Workflow Format)

```python
import requests
import base64
import json

# Read and encode your image
with open("character.png", "rb") as f:
    image_data = base64.b64encode(f.read()).decode("utf-8")

# Load workflow
with open("test_resources/workflows/workflow_character_dataset.json", "r") as f:
    workflow_data = json.load(f)

# Prepare request
payload = {
    "input": {
        "workflow": workflow_data["input"]["workflow"],
        "images": [
            {
                "name": "input_character.png",
                "image": f"data:image/png;base64,{image_data}"
            }
        ]
    }
}

# Make request
response = requests.post(
    f"https://api.runpod.ai/v2/{ENDPOINT_ID}/runsync",
    headers={
        "Authorization": f"Bearer {API_KEY}",
        "Content-Type": "application/json"
    },
    json=payload
)

# Get result
result = response.json()
output_image_base64 = result["output"]["images"][0]["data"]
```

### cURL Example (Minimal Format)

```bash
# Encode your image
IMAGE_BASE64=$(base64 -w 0 character.png)

# Send request
curl -X POST \
  -H "Authorization: Bearer ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d "{
    \"input\": {
      \"prompt\": \"Make this person standing in a beautiful garden\",
      \"image\": \"data:image/png;base64,${IMAGE_BASE64}\",
      \"seed\": 123456789,
      \"width\": 1024,
      \"height\": 1024
    }
  }" \
  https://api.runpod.ai/v2/${ENDPOINT_ID}/runsync
```

### cURL Example (Full Workflow Format)

```bash
# Encode your image
IMAGE_BASE64=$(base64 -w 0 character.png)

# Send request
curl -X POST \
  -H "Authorization: Bearer ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d "{
    \"input\": {
      \"workflow\": $(cat test_resources/workflows/workflow_character_dataset.json | jq '.input.workflow'),
      \"images\": [
        {
          \"name\": \"input_character.png\",
          \"image\": \"data:image/png;base64,${IMAGE_BASE64}\"
        }
      ]
    }
  }" \
  https://api.runpod.ai/v2/${ENDPOINT_ID}/runsync
```

## Performance Optimization

### Recommended GPU Configurations

- **Minimum**: RTX 4090 (24GB VRAM)
- **Recommended**: A100 (40GB VRAM)
- **Optimal**: H100 (80GB VRAM)

### Expected Generation Times

| GPU         | Resolution | Steps | Time    |
|-------------|------------|-------|---------|
| RTX 4090    | 2048x2048 | 4     | ~15s    |
| RTX 4090    | 3072x3072 | 4     | ~30s    |
| A100 40GB   | 2048x2048 | 4     | ~10s    |
| A100 40GB   | 3072x3072 | 4     | ~20s    |
| H100 80GB   | 3072x3072 | 4     | ~12s    |

## Related Documentation

- [Main README](../README.md)
- [Deployment Guide](deployment.md)
- [Configuration Guide](configuration.md)
- [Customization Guide](customization.md)

## Support

For issues or questions:

1. Check the [troubleshooting section](#troubleshooting) above
2. Review [ComfyUI documentation](https://github.com/comfyanonymous/ComfyUI)
3. Open an issue on the project repository
