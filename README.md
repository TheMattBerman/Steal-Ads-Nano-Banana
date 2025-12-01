# 🎯 Ad Intelligence System

**Automatically spy on competitor Facebook ads, analyze what makes them work, and generate on-brand creative alternatives using AI.**

This is a 4-workflow n8n system that watches competitor ads, uses AI vision to analyze their effectiveness, and generates branded alternatives—all automatically.

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![n8n](https://img.shields.io/badge/n8n-1.0+-orange.svg)
![AI](https://img.shields.io/badge/AI-Claude%20%7C%20Gemini-purple.svg)

---

## 🎬 What It Does

```
Competitor Facebook Ads → AI Analysis → Branded Creative Generation
```

1. **Collects** competitor ads from Facebook Ad Library via API
2. **Scores** ads by effectiveness (run duration × variation count)
3. **Analyzes** visuals with Gemini Vision (composition, colors, layout, text)
4. **Extracts** marketing patterns with Claude (hooks, angles, triggers, audience)
5. **Generates** on-brand copy and headlines with Claude
6. **Creates** new ad images with NanaBanana Pro (Gemini 3 Pro Image)
7. **Saves** everything to Google Sheets and Google Drive

---

## 📋 Table of Contents

- [Prerequisites](#-prerequisites)
- [Quick Start](#-quick-start)
- [Workflow Overview](#-workflow-overview)
- [Setup Instructions](#-setup-instructions)
- [Configuration](#-configuration)
- [Google Sheets Setup](#-google-sheets-setup)
- [API Keys Required](#-api-keys-required)
- [Customization](#-customization)
- [Troubleshooting](#-troubleshooting)
- [License](#-license)

---

## 🔧 Prerequisites

- **n8n** (self-hosted or cloud) - version 1.0+
- **OpenRouter account** - for AI models (Claude, Gemini, NanaBanana)
- **ScrapeCreators account** - for Facebook Ad Library API
- **Google Cloud account** - for Sheets and Drive APIs
- **Basic familiarity with n8n** - importing workflows, setting credentials

---

## 🚀 Quick Start

### 1. Clone this repo
```bash
git clone https://github.com/yourusername/ad-intelligence-system.git
cd ad-intelligence-system
```

### 2. Import workflows into n8n
1. Open your n8n instance
2. Go to **Workflows** → **Import from File**
3. Import each workflow JSON file in order:
   - `workflow-1-ad-collector.json`
   - `workflow-2-visual-analyzer.json`
   - `workflow-3-creative-generator.json`
   - `workflow-4-notifier.json` (optional)

### 3. Set up Google Sheets
1. Create a new Google Sheet
2. Create two tabs: `Competitor_Ads` and `Brand_Guidelines`
3. Copy the column headers from [Google Sheets Setup](#-google-sheets-setup)

### 4. Configure credentials
1. Set up each credential in n8n (see [API Keys Required](#-api-keys-required))
2. Update the Google Sheets document ID in each workflow

### 5. Configure your target competitor
1. Open Workflow 1
2. Find the "Config Settings" node
3. Update the `page_id` to your target competitor's Facebook Page ID

### 6. Activate and run!
1. Activate all workflows
2. Manually trigger Workflow 1 to test
3. Watch the magic happen ✨

---

## 🔄 Workflow Overview

### Workflow 1: Ad Collector & Differ
**Trigger:** Schedule (weekly) or Manual

**Purpose:** Fetch competitor ads and identify new/changed ones

**Key nodes:**
- Fetch ads from ScrapeCreators API
- Calculate effectiveness score
- Compare against existing ads (hash-based diffing)
- Filter ads with visuals only
- Trigger Workflow 2 for each new ad

**Effectiveness Formula:**
```
score = (runDurationDays × 0.7) + (collationCount × 10 × 0.3)
```

---

### Workflow 2: Visual Analyzer
**Trigger:** Execute Sub-Workflow (from Workflow 1)

**Purpose:** AI-powered deep analysis of ad creative

**Key nodes:**
- Download ad image
- **Gemini 3 Pro Vision** - Analyzes image structure:
  - Image quality, format, composition
  - Color palette with hex codes
  - Layout and visual hierarchy
  - Text/OCR extraction
  - Ad style indicators (UGC, testimonial, social proof)
- **Claude Sonnet 4** - Analyzes marketing strategy:
  - Hook type and effectiveness
  - Angle (educational, emotional, etc.)
  - Patterns (urgency, social proof, identity targeting)
  - Emotional triggers
  - Target audience signals
- Save analysis to Google Sheets
- Trigger Workflow 3

---

### Workflow 3: Creative Generator
**Trigger:** Execute Sub-Workflow (from Workflow 2)

**Purpose:** Generate branded alternatives to competitor ads

**Key nodes:**
- Load brand guidelines from Google Sheets
- Merge ad data with brand context
- **Claude Sonnet 4.5** - Generates creative package:
  - Headlines with rationale
  - Body copy with rationale
  - CTA with rationale
  - Visual direction
  - Detailed image prompt
- **NanaBanana Pro** - Generates ad image:
  - Uses brand colors and style
  - Matches competitor's aspect ratio
  - 2K resolution
- Upload image to Google Drive
- Update Google Sheets with generated content

---

### Workflow 4: Notifier (Optional)
**Trigger:** Execute Sub-Workflow (from Workflow 3)

**Purpose:** Alert when new creative is ready

**Key nodes:**
- Send Slack notification
- Include preview of generated creative

---

## ⚙️ Setup Instructions

### Step 1: Import Workflows

1. Download all workflow JSON files from the `/workflows` folder
2. In n8n, go to **Workflows** → **Import from File**
3. Import each file in order (1 → 2 → 3 → 4)
4. After importing, you'll see warning icons on nodes that need credentials

### Step 2: Set Up Credentials

In n8n, go to **Credentials** and create:

| Credential Type | Name Suggestion | Used In |
|----------------|-----------------|---------|
| Header Auth | OpenRouter API | Workflows 2, 3 |
| Header Auth | ScrapeCreators API | Workflow 1 |
| OAuth2 | Google Sheets | All workflows |
| OAuth2 | Google Drive | Workflow 3 |
| OAuth2 | Slack (optional) | Workflow 4 |

See [API Keys Required](#-api-keys-required) for details on each.

### Step 3: Configure Google Sheets

1. Create a new Google Spreadsheet
2. Copy the Document ID from the URL:
   ```
   https://docs.google.com/spreadsheets/d/[THIS-IS-THE-DOCUMENT-ID]/edit
   ```
3. Update this ID in all Google Sheets nodes across all workflows

### Step 4: Update Competitor Target

In Workflow 1, find the "Config Settings" node and update:
```javascript
const config = {
  competitor_page_id: "YOUR_COMPETITOR_PAGE_ID",  // Facebook Page ID
  competitor_name: "Competitor Name",
  // ... other settings
};
```

**How to find a Facebook Page ID:**
1. Go to the competitor's Facebook page
2. View page source or use a tool like [Find My FB ID](https://findmyfbid.com/)

### Step 5: Update Brand Guidelines

In your Google Sheet's `Brand_Guidelines` tab, update the values to match your brand:

| Field | Value |
|-------|-------|
| brand_name | Your Brand Name |
| tagline | Your tagline |
| primary_color | #EC4899 |
| secondary_color | #1F2937 |
| accent_color | #10B981 |
| voice | Direct, tactical, proof-driven |
| target_audience | Your target audience description |
| forbidden_words | game-changer, revolutionary, guru |

### Step 6: Test the System

1. Activate all workflows (toggle the switch in the top-right)
2. Open Workflow 1
3. Click "Execute Workflow" to run manually
4. Monitor the execution and check each workflow triggers correctly
5. Verify data appears in your Google Sheet

---

## 📊 Google Sheets Setup

### Tab 1: Competitor_Ads

Create these column headers in row 1:

| Column | Header | Description |
|--------|--------|-------------|
| A | ad_archive_id | Unique ad identifier |
| B | page_id | Facebook Page ID |
| C | page_name | Competitor name |
| D | start_date | When ad started running |
| E | end_date | When ad stopped (if applicable) |
| F | run_duration_days | Days the ad has been running |
| G | is_active | Whether ad is currently active |
| H | collation_count | Number of ad variations |
| I | effectiveness_score | Calculated performance score |
| J | body_text | Ad copy text |
| K | cta_type | Call-to-action type |
| L | media_type | Image, video, carousel |
| M | image_url | URL to ad image |
| N | video_url | URL to ad video (if applicable) |
| O | video_thumbnail_url | Video thumbnail URL |
| P | creative_hash | Hash for change detection |
| Q | last_checked | Last time ad was processed |
| R | status | Processing status |
| S | vision_json | Gemini vision analysis (JSON) |
| T | marketing_insights | Claude marketing analysis (JSON) |
| U | generated_headlines | AI-generated headlines |
| V | generated_body_copy | AI-generated body copy |
| W | generated_image_prompt | Prompt used for image generation |
| X | generated_image_url | Google Drive link to generated image |

### Tab 2: Brand_Guidelines

Create two columns - Field (A) and Value (B):

| Field | Value (customize these) |
|-------|------------------------|
| brand_name | Your Brand |
| tagline | Your tagline here |
| primary_color | #EC4899 |
| secondary_color | #1F2937 |
| accent_color | #10B981 |
| voice | Direct, tactical, proof-driven |
| target_audience | Your target audience |
| forbidden_words | game-changer, revolutionary, guru |
| logo_url | https://your-logo-url.com/logo.png |
| website | https://yourbrand.com |

---

## 🔑 API Keys Required

### 1. OpenRouter API
**Used for:** Claude Sonnet 4.5, Gemini 3 Pro Vision, NanaBanana Pro

1. Sign up at [openrouter.ai](https://openrouter.ai)
2. Go to API Keys and create a new key
3. Add credits to your account
4. In n8n, create a **Header Auth** credential:
   - Name: `Authorization`
   - Value: `Bearer YOUR_OPENROUTER_API_KEY`

**Models used:**
- `google/gemini-3-pro-preview` - Vision analysis
- `anthropic/claude-sonnet-4` - Marketing analysis & creative generation
- `google/gemini-3-pro-image-preview` - Image generation (NanaBanana Pro)

### 2. ScrapeCreators API
**Used for:** Facebook Ad Library data

1. Sign up at [scrapecreators.com](https://scrapecreators.com)
2. Get your API key from the dashboard
3. In n8n, create a **Header Auth** credential:
   - Name: `x-api-key`
   - Value: `YOUR_SCRAPECREATORS_API_KEY`

### 3. Google OAuth2
**Used for:** Google Sheets and Google Drive

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a new project (or use existing)
3. Enable **Google Sheets API** and **Google Drive API**
4. Create OAuth2 credentials (Desktop app type)
5. In n8n, create **Google Sheets OAuth2** and **Google Drive OAuth2** credentials
6. Complete the OAuth flow to authorize

### 4. Slack (Optional)
**Used for:** Notifications in Workflow 4

1. Create a Slack app at [api.slack.com](https://api.slack.com/apps)
2. Add OAuth scopes: `chat:write`, `files:write`
3. Install to your workspace
4. In n8n, create a **Slack OAuth2** credential

---

## 🎨 Customization

### Change the Competitor

Update the `page_id` in Workflow 1's Config Settings node:

```javascript
const config = {
  competitor_page_id: "NEW_PAGE_ID",
  competitor_name: "New Competitor Name",
};
```

### Add Multiple Competitors

Modify Workflow 1 to loop through multiple page IDs:

```javascript
const competitors = [
  { page_id: "123456789", name: "Competitor A" },
  { page_id: "987654321", name: "Competitor B" },
];
```

### Customize Brand Voice

Update your `Brand_Guidelines` tab with:
- Your brand colors (hex codes)
- Your voice/tone description
- Words to avoid
- Target audience description

### Change Image Aspect Ratio

The system automatically detects the competitor's aspect ratio. To override:

In Workflow 3's "Build Image Request" node, change:
```javascript
let aspectRatio = "1:1";  // Force square
// Options: "1:1", "4:5", "9:16", "16:9"
```

### Adjust Effectiveness Scoring

In Workflow 1, modify the effectiveness formula:
```javascript
const score = (runDurationDays * 0.7) + (collationCount * 10 * 0.3);
```

Increase `runDurationDays` weight if you value longevity more.
Increase `collationCount` weight if you value active testing more.

---

## 🐛 Troubleshooting

### "Node 'X' hasn't been executed"

This usually means you're referencing a node that isn't in the current execution path. Use:
```javascript
$('When Executed by Another Workflow').first().json
```
Instead of:
```javascript
items[0].json
```

### Images not generating correctly

1. Check the NanaBanana output - images are in `choices[0].message.images[0].image_url.url`
2. Ensure you're extracting base64 from the data URL: `dataUrl.split(',')[1]`
3. Verify the base64 starts with `/9j/` (JPEG) or `iVBORw` (PNG)

### Google Sheets not updating

1. Verify the Document ID is correct
2. Check that the tab names match exactly (`Competitor_Ads`, `Brand_Guidelines`)
3. Ensure the OAuth credential has write permissions
4. Check that "Always Output Data" is enabled on Google Sheets nodes

### Workflow not triggering next workflow

1. Ensure all workflows are **Active** (toggle in top-right)
2. Check that Execute Sub-Workflow nodes have the correct workflow selected
3. Verify the target workflow has a "When Executed by Another Workflow" trigger

### Vision analysis returning empty

1. Check that the image URL is accessible
2. Verify the image was downloaded successfully (check binary data)
3. Ensure OpenRouter API has sufficient credits

### Rate limits or timeouts

1. Add Wait nodes between API calls if hitting rate limits
2. For image generation (30-60 seconds), ensure your n8n timeout is sufficient
3. Consider adding retry logic for failed requests

---

## 📁 File Structure

```
ad-intelligence-system/
├── README.md
├── workflows/
│   ├── workflow-1-ad-collector.json
│   ├── workflow-2-visual-analyzer.json
│   ├── workflow-3-creative-generator.json
│   └── workflow-4-notifier.json
├── templates/
│   └── google-sheets-template.xlsx
└── examples/
    ├── sample-vision-output.json
    ├── sample-marketing-analysis.json
    └── sample-generated-creative.json
```

---

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- Built with [n8n](https://n8n.io) workflow automation
- AI powered by [Anthropic Claude](https://anthropic.com) and [Google Gemini](https://deepmind.google/technologies/gemini/)
- Ad data from [ScrapeCreators](https://scrapecreators.com)

---

## 📬 Support

- **Issues:** Open a GitHub issue for bugs or feature requests
- **Twitter:** Follow [@BigPlayersHQ](https://twitter.com/themattberman) for updates
- **Newsletter:** Subscribe to [Big Players](https://bigplayers.co) for growth strategies

---

Built with ❤️ by the Big Players team
