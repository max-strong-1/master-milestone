# Master Milestone

> **AI-powered voice commerce system for construction materials ordering**

A comprehensive monorepo containing multiple voice agent implementations, product management systems, and educational knowledge bases for Milestone Trucks - enabling customers to order gravel, soil, and mulch by voice across 7,802 ZIP codes in OH, IN, PA, WV, KY, and MI.

[![GitHub](https://img.shields.io/badge/GitHub-master--milestone-blue?logo=github)](https://github.com/max-strong-1/master-milestone)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

---

## 🎯 What This Repository Contains

This is a **complete voice commerce platform** with:

- **4 Voice Agent Implementations** - Next.js, Vercel Functions, and Flask/Python variants
- **Product Management System** - CSV processing, bulk updates, n8n workflows
- **Knowledge Base** - 40+ educational markdown files for AI training
- **Comprehensive Documentation** - Architecture guides, setup instructions, troubleshooting

---

## 🏗️ Repository Structure

```
Master Milestone/
├── Milestone Voice Agent/          # Voice agent implementations
│   ├── milestone-trucks-chat/      # Next.js + ElevenLabs (PRODUCTION)
│   ├── milestone-voice-agent/      # Standalone Vercel Functions
│   └── -milestone-voice-agent/     # Alternative Vercel implementation
│
├── Milestone Products/              # Product management
│   ├── elevenlabs-woocommerce-voice-agent/  # Flask/Python implementation
│   ├── -milestone-voice-agent/     # Additional Vercel functions
│   ├── CSV files/                  # Product databases
│   └── n8n workflows/              # Automated update pipelines
│
├── Milestone Rag (Knowledge Base)/ # AI training content
│   ├── 01_company/                 # Company info, brand voice
│   ├── 02_materials/               # 14 material definitions
│   ├── 03_project_guides/          # Driveways, drainage, etc.
│   ├── 04_calculations/            # Quantity formulas
│   ├── 05_delivery/                # Delivery logistics
│   ├── 06_customer_education/      # Educational content
│   ├── 07_objections/              # Objection handling
│   ├── 08_safety/                  # Safety warnings
│   └── 09_fallbacks/               # Error handling
│
├── Milestone Images/                # Image assets
├── CLAUDE.md                        # Technical documentation for AI agents
└── README.md                        # This file
```

---

## 🚀 Voice Agent Implementations

### 1. milestone-trucks-chat (Production - Next.js)

**Status:** ✅ Live Production System
**URL:** https://milestone-trucks-chat.vercel.app
**Platform:** Next.js 16.0.7 on Vercel

**Key Features:**
- Unified webhook endpoint for all server-side tools
- Client-side command queue for browser manipulation
- Integration with WooCommerce REST API
- ElevenLabs conversational AI integration

**Quick Start:**
```bash
cd " Milestone Voice Agent/milestone-trucks-chat"
npm install
npm run dev  # Start development server on localhost:3000
```

### 2. milestone-voice-agent (Vercel Functions)

**Status:** Alternative Implementation
**Platform:** Node.js Serverless Functions

**Key Features:**
- Individual API endpoints (not unified webhook)
- Educational material recommendations
- WooCommerce integration
- Health monitoring and cache management

**Quick Start:**
```bash
cd " Milestone Voice Agent/milestone-voice-agent"
npm install
npm run dev  # Start Vercel dev server
```

### 3. elevenlabs-woocommerce-voice-agent (Flask/Python)

**Status:** Python Alternative
**Platform:** Flask on Railway/Render

**Key Features:**
- Flask webhook server
- ZIP code validation and region mapping
- Product consultant with AI recommendations
- Command queue for client-side actions

**Quick Start:**
```bash
cd "Milestone Products /elevenlabs-woocommerce-voice-agent"
pip install -r server/requirements.txt
python server/app.py
```

---

## 📊 Key Data

### Service Area
- **7,802 ZIP codes** across 6 states
- States: Ohio, Indiana, Pennsylvania, West Virginia, Kentucky, Michigan

### Materials Catalog
- **14 construction materials** (Product IDs 101-114)
- Includes: Limestone, gravel, sand, mulch, topsoil
- **$75 flat delivery fee**

### Voice Agent: Robert
- **Personality:** Blue-collar foreman - patient, friendly, knowledgeable
- **Conversation Flow:** ZIP validation → Project discovery → Material recommendations → Quantity calculation → Checkout
- **Pronunciation Rules:** #57 = "fifty-seven", #304 = "three-oh-four"

---

## 🛠️ Technology Stack

| Component | Technology |
|-----------|-----------|
| **Voice AI** | ElevenLabs Conversational AI |
| **Frontend** | Next.js 16, React 19 |
| **Backend** | Vercel Serverless Functions, Flask |
| **E-commerce** | WooCommerce REST API |
| **Automation** | n8n workflows |
| **Deployment** | Vercel, Railway, Render |

---

## 📖 Documentation

### For Developers
- **[CLAUDE.md](CLAUDE.md)** - Comprehensive technical documentation for AI agents
- **Architecture Guide** - `Milestone Products /Milestone Trucks Voice Agent - Tool Architecture Documentation.md`
- **System Prompts** - See individual implementation directories

### For ElevenLabs Setup
- **milestone-trucks-chat**: See `ROBERT_SYSTEM_PROMPT.md`
- **milestone-voice-agent**: See `docs/SYSTEM_PROMPT.md` and `ELEVENLABS_WEBHOOK_SETUP.md`
- **Flask implementation**: See `docs/ELEVENLABS_SETUP.md`

---

## 🔑 Critical Architecture: Webhook vs Client Tools

The production system (milestone-trucks-chat) uses two distinct tool types:

**WEBHOOK TOOLS** (Server-side):
- Execute on Vercel backend
- Purpose: Database queries, calculations, validation
- Tools: `check_service_area`, `get_materials_by_zip`, `get_material_details`, `calculate_quantity`

**CLIENT TOOLS** (Browser-side):
- Execute in user's browser via command queue
- Purpose: Cart updates, navigation, form filling
- Tools: `create_or_update_cart`, `navigate_to`, `prefill_checkout_form`

⚠️ **CRITICAL:** These tool types are NOT interchangeable. See CLAUDE.md for details.

---

## ⚙️ Environment Variables

All implementations require WooCommerce API credentials:

```bash
WOOCOMMERCE_URL=https://staging12.milestonetrucks.com
WOOCOMMERCE_CONSUMER_KEY=ck_xxxxxxxxxxxxx
WOOCOMMERCE_CONSUMER_SECRET=cs_xxxxxxxxxxxxx
```

Get API keys: **WooCommerce → Settings → Advanced → REST API**

---

## 🧪 Testing

### Test Service Area Validation
```bash
curl -X POST https://milestone-trucks-chat.vercel.app/api/elevenlabs/webhook \
  -H "Content-Type: application/json" \
  -d '{"tool_name": "check_service_area", "zip_code": "43560"}'
```

### Test Quantity Calculation
```bash
curl -X POST https://milestone-trucks-chat.vercel.app/api/elevenlabs/webhook \
  -H "Content-Type: application/json" \
  -d '{"tool_name": "calculate_quantity", "length": 20, "width": 30, "depth": 4}'
```

### End-to-End Voice Test
1. Say: "Hi Robert, I'm in ZIP 43560"
2. Say: "I'm building a driveway"
3. Say: "It's 50 feet long and 12 feet wide, 4 inches deep"
4. Complete checkout flow

---

## 📈 Performance Metrics

From production testing:
- **LLM Processing:** ~17s
- **Text-to-Speech:** ~225ms
- **RAG Retrieval:** ~344ms
- **Tool Execution:** 1-259ms

---

## 🔧 Common Development Tasks

### Add a New ZIP Code
1. Edit `lib/service-area-zips.ts`
2. Add ZIP to `SERVICED_ZIP_CODES` array
3. Rebuild and redeploy

### Add a New Material
1. Add product to WooCommerce (Product ID 101-114)
2. Create markdown in `Milestone Rag (Knowledge Base)/02_materials/`
3. Update `lib/materials.ts` if using hardcoded definitions

### Update Agent Personality
Edit the system prompt file, then update in ElevenLabs dashboard

---

## 🐛 Troubleshooting

### Error: Invalid discriminator value (ElevenLabs)
**Solution:** Remove outer `tool_config` wrapper, use flat JSON structure

### Error: Tool not executing
**Solution:** Verify tool type matches function (data = webhook, UI = client)

### Products not found for ZIP
**Solution:** Add ZIP to `lib/service-area-zips.ts`

See [CLAUDE.md](CLAUDE.md) for comprehensive troubleshooting guide.

---

## 📞 Contact

- **Developer:** Kel Mewbourne
- **Company:** Foreman AI
- **Email:** maxstrongperformance@gmail.com
- **Phone:** 561-831-9310

---

## 📄 License

MIT License - See LICENSE file

---

## 🙏 Acknowledgments

Built with:
- [ElevenLabs Conversational AI](https://elevenlabs.io/)
- [WooCommerce REST API](https://woocommerce.github.io/woocommerce-rest-api-docs/)
- [Next.js](https://nextjs.org/)
- [Vercel](https://vercel.com/)
- [Flask](https://flask.palletsprojects.com/)

---

**Repository:** https://github.com/max-strong-1/master-milestone

*Last Updated: December 2024*
