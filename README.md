# Qwen Image Edit for RunPod Serverless
[한국어 README 보기](README_kr.md)

This project is a template designed to easily deploy and use an image editing workflow (Qwen Image Edit via ComfyUI) in the RunPod Serverless environment.

[![Runpod](https://api.runpod.io/badge/wlsdml1114/qwen_image_edit)](https://console.runpod.io/hub/wlsdml1114/qwen_image_edit)

The template performs prompt-guided image editing using ComfyUI workflows. It supports one, two, or three input images and accepts inputs as path, URL, or Base64.

## 🎨 Engui Studio Integration

[![EnguiStudio](https://raw.githubusercontent.com/wlsdml1114/Engui_Studio/main/assets/banner.png)](https://github.com/wlsdml1114/Engui_Studio)

This Qwen Image Edit template is primarily designed for **Engui Studio**, a comprehensive AI model management platform. While it can be used via API, Engui Studio provides enhanced features and broader model support.

**Engui Studio Benefits:**
- **Expanded Model Support**: Access to a wide variety of AI models beyond what's available through API
- **Enhanced User Interface**: Intuitive workflow management and model selection
- **Advanced Features**: Additional tools and capabilities for AI model deployment
- **Seamless Integration**: Optimized for Engui Studio's ecosystem

> **Note**: While this template works perfectly with API calls, Engui Studio users will have access to additional models and features that are planned for future releases.

## ✨ Key Features

*   **Prompt-Guided Image Editing**: Edit images based on a text prompt.
*   **One, Two, or Three Input Images**: Automatically selects the workflow by the number of images (1/2/3).
*   **Flexible Inputs**: Provide images via file path, URL, or Base64 string.
*   **Customizable Parameters**: Control seed, width, height, and prompt.
*   **ComfyUI Integration**: Built on top of ComfyUI for flexible workflow management.

## 🚀 RunPod Serverless Template

This template includes all the necessary components to run Qwen Image Edit as a RunPod Serverless Worker.

*   **Dockerfile**: Configures the environment and installs all dependencies required for model execution.
*   **handler.py**: Implements the handler function that processes requests for RunPod Serverless.
*   **entrypoint.sh**: Performs initialization tasks when the worker starts.
*   **qwen_image_edit_1_1image.json / qwen_image_edit_1_2image.json / qwen_image_edit_1_3image.json**: ComfyUI workflows for 1-, 2-, or 3-image editing.

### Input

The `input` object must contain the following fields. Image inputs support **URL, file path, or Base64 encoded string**.

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `prompt` | `string` | **Yes** | `N/A` | Text prompt that guides the edit. |
| `image_path` or `image_url` or `image_base64` | `string` | **Yes** | `N/A` | First image input (path/URL/Base64). |
| `image_path_2` or `image_url_2` or `image_base64_2` | `string` | No | `N/A` | Optional second image input (path/URL/Base64). |
| `image_path_3` or `image_url_3` or `image_base64_3` | `string` | No | `N/A` | Optional third image input (path/URL/Base64). |
| `lora_repo` | `string` | No | `N/A` | Hugging Face repository ID for a custom LoRA (e.g., `username/repo`). |
| `lora_scale` | `float` | No | `1.0` | Strength of the LoRA effect. |
| `seed` | `integer` | **Yes** | `N/A` | Random seed for deterministic output. |
| `width` | `integer` | **Yes** | `N/A` | Output image width in pixels. |
| `height` | `integer` | **Yes** | `N/A` | Output image height in pixels. |
| `redirect_url` | `boolean` | No | `false` | If `true`, upload result to R2 and return `image_url`; otherwise return Base64 `image`. |

Notes:
- Guidance is not used by the current handler.
- The workflow is selected automatically by the number of images provided (1, 2, or 3).

**Request Example (single image via URL):**

```json
{
  "input": {
    "prompt": "add watercolor style, soft pastel tones",
    "image_url": "https://path/to/your/reference.jpg",
    "seed": 12345,
    "width": 768,
    "height": 1024
  }
}
```

**Request Example (dual images, path + URL):**

```json
{
  "input": {
    "prompt": "blend subject A and subject B, cinematic lighting",
    "image_path": "/network_volume/img_a.jpg",
    "image_url_2": "https://path/to/img_b.jpg",
    "seed": 7777,
    "width": 1024,
    "height": 1024
  }
}
```

**Request Example (single image via Base64):**

```json
{
  "input": {
    "prompt": "vintage look, grain, warm tones",
    "image_base64": "<BASE64_STRING>",
    "seed": 42,
    "width": 512,
    "height": 512
  }
}
```

### Output

#### Success

If the job is successful:

- **Default** (`redirect_url` not set or `false`): returns JSON with Base64-encoded image.
- **With `redirect_url: true`**: uploads the image to Cloudflare R2 and returns the public/presigned URL (requires `R2_*` env vars).

| Parameter | Type | Description |
| --- | --- | --- |
| `image` | `string` | Base64-encoded image data (when `redirect_url` is not true). |
| `image_url` | `string` | Public or presigned URL of the image on R2 (when `redirect_url: true`). |

**Success Response Example (Base64):**

```json
{
  "image": "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg=="
}
```

**Success Response Example (R2 URL, when `redirect_url: true`):**

```json
{
  "image_url": "https://pub-xxx.r2.dev/temporary/task_xxx.png"
}
```

#### Error

If the job fails, it returns a JSON object containing an error message.

| Parameter | Type | Description |
| --- | --- | --- |
| `error` | `string` | Description of the error that occurred. |

**Error Response Example:**

```json
{
  "error": "이미지를 찾을 수 없습니다."
}
```

## 🛠️ Usage and API Reference

1.  Create a Serverless Endpoint on RunPod based on this repository.
2.  Once the build is complete and the endpoint is active, submit jobs via HTTP POST requests according to the API Reference above.

### API test script

From the project root, you can run the API test script (uses RunPod `/runsync`). Set `runpod_API_KEY` and `qwen_image_edit` (endpoint ID) in the project root `test.env`, or export them.

Input image: `qwen_edit/examples/input/test_input.png`. Outputs are written to `qwen_edit/examples/output/` by default (use `--out` to override).

```bash
# Recommended: use local example image (avoids external URL rate limits like HTTP 429)
python qwen_edit/test_api.py --mode base64

# S3(Network Volume) upload + image_path test (requires boto3: pip install boto3)
python qwen_edit/test_api.py --mode s3

# Run both base64 + S3 tests sequentially → examples/output/out_test.png, out_test_s3.png
python qwen_edit/test_api.py --all

# Using a JSON input file
python qwen_edit/test_api.py --json qwen_edit/example_request.json

# (Optional) URL mode (may fail if the host blocks automated downloads)
python qwen_edit/test_api.py --mode url --image-url "https://example.com/your-image.jpg"
```

Optional: `TEST_IMAGE_URL` in `test.env` can be used instead of `--image-url`. See `qwen_edit/.env.example` for a template without personal data.

### 📁 Using Network Volumes & Dynamic LoRA

Instead of directly transmitting Base64 encoded files, you can use RunPod's Network Volumes to handle large files and cache models.

1.  **Create and Connect Network Volume**: Create a Network Volume (e.g., 20GB+) from the RunPod dashboard and connect it to your Serverless Endpoint settings.
2.  **Mount Path**: Set the mount path to `/runpod-volume`.
3.  **Dynamic LoRA Caching**: When you provide a `lora_repo`, the worker will automatically download the LoRA from Hugging Face and store it in `/runpod-volume/loras`. Subsequent requests using the same LoRA will load instantly from the volume.
4.  **Private LoRAs**: To use private Hugging Face repositories, set the `HF_TOKEN` environment variable in your RunPod Endpoint settings.
5.  **Upload Files**: Upload the image files you want to use to the created Network Volume.
6.  **Specify Paths**: When making an API request, specify the file paths within the Network Volume for `image_path` or `image_path_2`. For example, if the volume is mounted at `/runpod-volume` and you use `reference.jpg`, the path would be `"/runpod-volume/reference.jpg"`.

### R2 (redirect_url)

When `redirect_url: true` is set in the request input, the handler uploads the generated image to Cloudflare R2 and returns `image_url`. Set these environment variables in your RunPod Endpoint:

| Variable | Required | Description |
| --- | --- | --- |
| `R2_ACCOUNT_ID` | Yes | R2 account ID. |
| `R2_ACCESS_KEY_ID` | Yes | R2 access key. |
| `R2_SECRET_ACCESS_KEY` | Yes | R2 secret key. |
| `R2_BUCKET_NAME` | Yes | Bucket name. |
| `R2_PUBLIC_URL` or `R2_CUSTOM_DOMAIN` | No | Public base URL for images (e.g. `https://pub-xxx.r2.dev`). If unset, a presigned URL (1h) is returned. |

### Example request files

Example request bodies (no personal data) are provided for copy-paste or use with `test_api.py --json`:

*   **example_request.json**: Single image via URL
*   **example_request_2images.json**: Two images (path + URL)
*   **example_request_3images.json**: Three images via URL

Copy `.env.example` to set `runpod_API_KEY` and `qwen_image_edit` for local testing.

## 🔧 Workflow Configuration

This template includes the following workflow configurations:

*   **qwen_image_edit_1_1image.json**: Single-image editing workflow
*   **qwen_image_edit_1_2image.json**: Two-image editing workflow
*   **qwen_image_edit_1_3image.json**: Three-image editing workflow

The workflows are based on ComfyUI and include necessary nodes for prompt-guided image editing and output processing.

## 🙏 Original Project

This project is based on the following repositories. All rights to the model and core logic belong to the original authors.

*   **ComfyUI:** [https://github.com/comfyanonymous/ComfyUI](https://github.com/comfyanonymous/ComfyUI)
*   **Qwen (project group):** [https://github.com/QwenLM/Qwen-Image](https://github.com/QwenLM/Qwen-Image)

## 📄 License

This template adheres to the licenses of the original projects.
