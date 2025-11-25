# HunyuanVideo API Reference

## Base URL

```
http://localhost:8000  # Local deployment
http://YOUR_SERVER_IP:8000  # Cloud deployment
```

---

## Endpoints

### 1. Generate Video

**POST** `/generate`

Generate a video from text prompt.

**Request Body:**
```json
{
  "prompt": "Professional educational video about data privacy principles",
  "duration": 10,
  "resolution": "1080p",
  "model": "HunyuanVideo-1.5"
}
```

**Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `prompt` | string | Yes | - | Text description of desired video |
| `duration` | integer | No | 10 | Video length in seconds (5-10) |
| `resolution` | string | No | "1080p" | Output resolution ("720p", "1080p") |
| `model` | string | No | "HunyuanVideo-1.5" | Model version to use |

**Response:**
```json
{
  "task_id": "uuid-1234-5678",
  "status": "processing",
  "estimated_time": 45
}
```

---

### 2. Check Status

**GET** `/status/{task_id}`

Check the status of a video generation task.

**Response (Processing):**
```json
{
  "task_id": "uuid-1234-5678",
  "status": "processing",
  "progress": 65,
  "elapsed_time": 30
}
```

**Response (Completed):**
```json
{
  "task_id": "uuid-1234-5678",
  "status": "completed",
  "video_url": "http://localhost:8000/videos/uuid-1234-5678.mp4",
  "duration": 10,
  "resolution": "1080p",
  "file_size": 15728640
}
```

**Response (Failed):**
```json
{
  "task_id": "uuid-1234-5678",
  "status": "failed",
  "error": "Out of memory",
  "error_code": "OOM_ERROR"
}
```

---

### 3. Download Video

**GET** `/videos/{task_id}.mp4`

Download the generated video file.

**Response:** Binary video file (MP4 format)

---

### 4. Health Check

**GET** `/health`

Check if the API server is running.

**Response:**
```json
{
  "status": "healthy",
  "model_loaded": true,
  "gpu_available": true,
  "vram_free": "18.5 GB",
  "version": "1.5.0"
}
```

---

## Status Codes

| Code | Status | Description |
|------|--------|-------------|
| 200 | OK | Request successful |
| 202 | Accepted | Task queued for processing |
| 400 | Bad Request | Invalid parameters |
| 404 | Not Found | Task ID not found |
| 500 | Internal Server Error | Server error |
| 503 | Service Unavailable | Model not loaded or GPU unavailable |

---

## Error Codes

| Error Code | Description | Solution |
|------------|-------------|----------|
| `INVALID_PROMPT` | Prompt is empty or too long | Provide 10-500 character prompt |
| `INVALID_DURATION` | Duration out of range | Use 5-10 seconds |
| `OOM_ERROR` | Out of GPU memory | Reduce resolution or duration |
| `MODEL_NOT_LOADED` | Model not initialized | Wait for model to load |
| `TASK_NOT_FOUND` | Task ID doesn't exist | Check task ID |
| `GENERATION_FAILED` | Video generation failed | Check logs for details |

---

## Rate Limits

- **Concurrent requests:** 1 (single GPU)
- **Queue size:** 10 tasks
- **Max prompt length:** 500 characters
- **Max duration:** 10 seconds

---

## Examples

### cURL

```bash
# Generate video
curl -X POST http://localhost:8000/generate \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Educational video about GDPR compliance",
    "duration": 10,
    "resolution": "1080p"
  }'

# Check status
curl http://localhost:8000/status/uuid-1234-5678

# Download video
curl -O http://localhost:8000/videos/uuid-1234-5678.mp4
```

### Python

```python
import requests
import time

# Generate video
response = requests.post(
    "http://localhost:8000/generate",
    json={
        "prompt": "Professional training video about data privacy",
        "duration": 10,
        "resolution": "1080p"
    }
)
task_id = response.json()["task_id"]

# Poll for completion
while True:
    status_response = requests.get(f"http://localhost:8000/status/{task_id}")
    status = status_response.json()
    
    if status["status"] == "completed":
        video_url = status["video_url"]
        print(f"Video ready: {video_url}")
        break
    elif status["status"] == "failed":
        print(f"Generation failed: {status['error']}")
        break
    
    time.sleep(5)

# Download video
video_response = requests.get(video_url)
with open("output.mp4", "wb") as f:
    f.write(video_response.content)
```

### JavaScript (Node.js)

```javascript
const axios = require('axios');
const fs = require('fs');

async function generateVideo() {
  // Generate video
  const response = await axios.post('http://localhost:8000/generate', {
    prompt: 'Educational video about cybersecurity basics',
    duration: 10,
    resolution: '1080p'
  });
  
  const taskId = response.data.task_id;
  
  // Poll for completion
  while (true) {
    const statusResponse = await axios.get(`http://localhost:8000/status/${taskId}`);
    const status = statusResponse.data;
    
    if (status.status === 'completed') {
      console.log(`Video ready: ${status.video_url}`);
      
      // Download video
      const videoResponse = await axios.get(status.video_url, {
        responseType: 'arraybuffer'
      });
      fs.writeFileSync('output.mp4', videoResponse.data);
      break;
    } else if (status.status === 'failed') {
      console.error(`Generation failed: ${status.error}`);
      break;
    }
    
    await new Promise(resolve => setTimeout(resolve, 5000));
  }
}

generateVideo();
```

---

## Prompt Engineering Tips

### Good Prompts

✅ **Specific and descriptive:**
```
"Professional educational video explaining GDPR Article 5 principles with clean text overlays and modern corporate style"
```

✅ **Include style keywords:**
```
"Animated explainer video about data breach notification requirements, minimalist design, smooth transitions"
```

✅ **Mention visual elements:**
```
"Training video showing PMP project lifecycle phases with diagrams, charts, and professional narration style"
```

### Bad Prompts

❌ **Too vague:**
```
"Privacy video"
```

❌ **Too long:**
```
"Create a comprehensive detailed video that thoroughly explains every single aspect of..."
```

❌ **Conflicting instructions:**
```
"Fast-paced slow-motion video with loud quiet music"
```

---

## Performance Optimization

### GPU Memory Management

| Resolution | Duration | VRAM Required | Generation Time |
|------------|----------|---------------|-----------------|
| 720p | 5s | ~12 GB | 20-30s |
| 720p | 10s | ~14 GB | 40-60s |
| 1080p | 5s | ~16 GB | 30-45s |
| 1080p | 10s | ~20 GB | 60-90s |

### Batch Processing

For multiple videos, process sequentially to avoid OOM errors:

```python
prompts = [
    "Video 1 prompt",
    "Video 2 prompt",
    "Video 3 prompt"
]

for prompt in prompts:
    generate_video(prompt)
    # Wait for completion before next
```

---

## Troubleshooting

### Common Issues

**Issue:** "Model not loaded"
- **Solution:** Wait 2-3 minutes after server start for model initialization

**Issue:** "Out of memory"
- **Solution:** Reduce resolution or duration, or restart server

**Issue:** "Generation taking too long"
- **Solution:** Check GPU utilization with `nvidia-smi`, ensure no other processes using GPU

**Issue:** "Video quality poor"
- **Solution:** Improve prompt specificity, use longer duration, increase resolution

---

## Advanced Configuration

### Environment Variables

```bash
# Server configuration
export HUNYUAN_HOST=0.0.0.0
export HUNYUAN_PORT=8000
export HUNYUAN_WORKERS=1

# Model configuration
export HUNYUAN_MODEL_PATH=/path/to/models
export HUNYUAN_CACHE_DIR=/path/to/cache
export HUNYUAN_MAX_QUEUE_SIZE=10

# Performance tuning
export HUNYUAN_ENABLE_FLASH_ATTN=true
export HUNYUAN_ENABLE_SAGE_ATTN=false
export HUNYUAN_OFFLOAD_CPU=false
```

### Custom API Server

See HunyuanVideo repository for full API server implementation:
https://github.com/Tencent-Hunyuan/HunyuanVideo-1.5

---

*Last Updated: November 25, 2025*
