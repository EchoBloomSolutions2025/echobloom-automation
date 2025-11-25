# EchoBloom AI Video Automation System - Complete Setup Guide

## 🎯 System Overview

This automation system generates AI-powered certification training videos using open-source models, with zero ongoing API costs. Content flows from Airtable → n8n → HunyuanVideo → Google Drive automatically.

---

## 📋 Architecture

```
Airtable (Content Queue)
    ↓
n8n Workflow Engine
    ↓
HunyuanVideo-1.5 (Self-Hosted AI)
    ↓
Google Drive (Storage)
    ↓
Distribution (TikTok, Gumroad, etc.)
```

---

## 🚀 Phase 1: Self-Host HunyuanVideo AI Model

### Option A: Local GPU (14+ GB VRAM)

**Requirements:**
- NVIDIA GPU with 14+ GB VRAM (RTX 3090, 4090, A4000, etc.)
- Ubuntu 22.04 or later
- Python 3.10+
- CUDA 11.8+

**Installation Steps:**

```bash
# 1. Clone HunyuanVideo repository
git clone https://github.com/Tencent-Hunyuan/HunyuanVideo-1.5.git
cd HunyuanVideo-1.5

# 2. Install dependencies
pip install -r requirements.txt
pip install -i https://mirrors.tencent.com/pypi/simple/ --upgrade tencentcloud-sdk-python

# 3. Install Flash Attention (for speed)
pip install flash-attn --no-build-isolation

# 4. Download model weights
python scripts/download_models.py

# 5. Start API server
python api_server.py --host 0.0.0.0 --port 8000
```

**Test the API:**
```bash
curl -X POST http://localhost:8000/generate \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Professional educational video about data privacy",
    "duration": 10,
    "resolution": "1080p"
  }'
```

### Option B: Cloud GPU (RunPod/Vast.ai)

**Cost:** ~$0.20-$0.80/hour (only when generating)

**RunPod Setup:**
1. Sign up at runpod.io
2. Deploy "PyTorch" template with 24GB+ VRAM
3. SSH into instance
4. Follow same installation steps as Option A
5. Expose port 8000 via RunPod proxy

**Vast.ai Setup:**
1. Sign up at vast.ai
2. Search for GPU with 24GB+ VRAM, sort by price
3. Rent instance with Ubuntu 22.04
4. SSH and install HunyuanVideo
5. Use Vast.ai's public IP for API access

---

## 🗄️ Phase 2: Set Up Airtable Content Database

### Create Airtable Base

1. **Sign in to Airtable:** airtable.com
2. **Create new base:** "EchoBloom Content Pipeline"
3. **Import schema:** Use `/airtable-schemas/content-queue-schema.json`

### Manual Setup (if import fails):

**Table: Content Queue**

| Field Name | Type | Options |
|------------|------|---------|
| Content ID | Auto Number | - |
| Certification | Single Select | CIPP/US, CIPM, CIPT, PMP, CISSP |
| Content Type | Single Select | Study Guide, Practice Questions, Concept Explanation |
| Topic | Single Line Text | - |
| Duration | Number | Default: 10 |
| Priority | Single Select | High, Medium, Low |
| Status | Single Select | Queued, Processing, Completed, Failed |
| Video URL | URL | - |
| Created Date | Date Time | - |
| Completed Date | Date Time | - |

### Get Airtable API Credentials

1. Go to airtable.com/account
2. Generate Personal Access Token
3. Copy Base ID from URL: `airtable.com/[BASE_ID]/[TABLE_ID]`
4. Save for n8n configuration

---

## ⚙️ Phase 3: Deploy n8n Automation Platform

### Option A: Self-Hosted (Free)

```bash
# Install n8n via Docker
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

Access at: http://localhost:5678

### Option B: n8n Cloud (Paid, Easier)

1. Sign up at n8n.io/cloud
2. Choose Starter plan ($20/month)
3. Skip Docker setup

### Import Workflow

1. Open n8n dashboard
2. Click "Import from File"
3. Upload `/n8n-workflows/ai-video-generation-workflow.json`
4. Configure credentials (see below)

### Configure Credentials

**Airtable:**
- Personal Access Token: [from Phase 2]
- Base ID: [from Phase 2]

**Google Drive:**
- OAuth2: Connect your Google account
- Folder ID: Create "EchoBloom Videos" folder, copy ID from URL

**HTTP Request (HunyuanVideo API):**
- Method: Header Auth
- Header Name: `Authorization`
- Header Value: `Bearer YOUR_API_KEY` (if you added auth)
- Or leave blank if no auth on local API

### Set Environment Variables

In n8n Settings → Environment Variables:

```
HUNYUAN_VIDEO_API_ENDPOINT=http://YOUR_SERVER_IP:8000
GOOGLE_DRIVE_FOLDER_ID=YOUR_FOLDER_ID
AIRTABLE_WATCH_BASE=YOUR_BASE_ID
```

---

## 📦 Phase 4: Set Up GitHub Repository

### Create Repository

```bash
cd /home/ubuntu/echobloom-automation

# Initialize git
git init
git add .
git commit -m "Initial commit: AI video automation system"

# Create GitHub repo
gh repo create echobloom-automation --public --source=. --remote=origin --push
```

### Add GitHub Secrets

Go to GitHub repo → Settings → Secrets and variables → Actions

Add these secrets:
- `N8N_API_URL`: Your n8n instance URL
- `N8N_API_KEY`: n8n API key (Settings → API)
- `HUNYUAN_API_ENDPOINT`: Your HunyuanVideo API URL

### Enable GitHub Actions

1. Go to Actions tab
2. Enable workflows
3. Push changes to trigger deployment

---

## 🎬 Phase 5: Test the Complete Pipeline

### Add Test Content to Airtable

1. Open Airtable base
2. Add new record:
   - **Certification:** CIPP/US
   - **Content Type:** Study Guide
   - **Topic:** Introduction to Privacy Principles
   - **Duration:** 10
   - **Priority:** High
   - **Status:** Queued

### Watch the Magic Happen

1. **n8n detects** new Airtable record
2. **Generates prompt** based on content type
3. **Calls HunyuanVideo API** to generate video
4. **Polls for completion** (30-60 seconds)
5. **Downloads video** when ready
6. **Uploads to Google Drive**
7. **Updates Airtable** with video URL and "Completed" status

### Check Results

- **Airtable:** Status should change to "Completed"
- **Google Drive:** Video should appear in folder
- **n8n:** Check execution log for details

---

## 💰 Cost Analysis

### Zero-Cost Setup (Local GPU)
- **Hardware:** Use existing GPU or one-time purchase
- **HunyuanVideo:** Free (open source)
- **n8n:** Free (self-hosted)
- **Airtable:** Free tier (1,200 records)
- **Google Drive:** Free (15 GB)
- **GitHub:** Free (public repos)

**Total Monthly Cost:** $0

### Low-Cost Setup (Cloud GPU)
- **RunPod GPU:** $0.20-$0.80/hour (only when generating)
- **n8n Cloud:** $20/month (optional, easier)
- **Airtable Pro:** $20/month (optional, more records)
- **Google Workspace:** $6/month (optional, more storage)

**Total Monthly Cost:** $20-$46 + GPU usage

### Example Usage Costs:
- **10 videos/day:** ~30 minutes GPU time = $0.10-$0.40/day = $3-$12/month
- **50 videos/day:** ~2.5 hours GPU time = $0.50-$2.00/day = $15-$60/month

---

## 🔧 Troubleshooting

### HunyuanVideo API Not Responding
```bash
# Check if server is running
curl http://localhost:8000/health

# Check logs
tail -f hunyuan_api.log

# Restart server
pkill -f api_server.py
python api_server.py --host 0.0.0.0 --port 8000
```

### n8n Workflow Failing
1. Check n8n execution logs
2. Verify all credentials are configured
3. Test each node individually
4. Check environment variables

### Airtable Not Triggering
1. Verify Airtable trigger is active
2. Check webhook URL is accessible
3. Test with manual execution first

### Google Drive Upload Failing
1. Re-authenticate Google OAuth
2. Check folder permissions
3. Verify folder ID is correct

---

## 📈 Scaling Up

### Batch Processing
- Add multiple records to Airtable at once
- n8n will process them sequentially
- For parallel processing, deploy multiple n8n instances

### Multiple Video Models
- Add Replicate API as fallback
- Use different models for different content types
- Implement model selection in Airtable

### Content Distribution
- Add TikTok upload node to workflow
- Auto-post to YouTube Shorts
- Schedule releases via Airtable

---

## 🎯 Next Steps

1. ✅ **Complete this setup guide**
2. **Generate 10 test videos** to validate pipeline
3. **Create content calendar** in Airtable (50-100 videos)
4. **Set up distribution** (TikTok, Gumroad)
5. **Monitor and optimize** (video quality, generation time)
6. **Scale production** (100+ videos/week)

---

## 📞 Support Resources

- **HunyuanVideo Docs:** https://github.com/Tencent-Hunyuan/HunyuanVideo-1.5
- **n8n Community:** https://community.n8n.io
- **Airtable Support:** https://support.airtable.com
- **GitHub Issues:** https://github.com/EchoBloomSolutions2025/echobloom-automation/issues

---

## 🏆 Success Metrics

Track these KPIs in Airtable:
- Videos generated per day
- Average generation time
- Success rate (completed vs failed)
- Cost per video
- Distribution reach (views, engagement)

**Target:** 50 videos/week at <$0.50/video cost

---

*Last Updated: November 25, 2025*
*Version: 1.0*
