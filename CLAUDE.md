# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

**Master Milestone** is a monorepo containing multiple voice agent implementations and supporting systems for Milestone Trucks, a construction materials delivery business serving 7,802 ZIP codes across OH, IN, PA, WV, KY, and MI.

### Core Systems

1. **Voice Agent Implementations** (4 variants)
   - milestone-trucks-chat (Next.js + ElevenLabs) - Production
   - milestone-voice-agent (Vercel Functions) - 2 copies
   - elevenlabs-woocommerce-voice-agent (Flask/Python)
2. **Product Management** - CSV processing, bulk updates, n8n workflows
3. **Knowledge Base** - Educational content for AI training (40+ markdown files)

## Voice Agent Implementations

### 1. milestone-trucks-chat (Next.js - Production System)

**Location**: ` Milestone Voice Agent/milestone-trucks-chat/`
**Live URL**: https://milestone-trucks-chat.vercel.app
**GitHub**: https://github.com/max-strong-1/milestone-trucks-chat
**Platform**: Next.js 16.0.7 on Vercel
**Status**: Production deployment

#### Critical Architecture: Webhook vs Client Tools

This is the KEY architectural distinction that cannot be violated:

**WEBHOOK TOOLS** = Server-side data operations:
- Execute on Vercel backend
- Endpoint: `app/api/elevenlabs/webhook/route.ts`
- Handler: Single webhook endpoint at `/api/elevenlabs/webhook`
- Purpose: Database queries, calculations, validation logic
- Tools: `check_service_area`, `get_materials_by_zip`, `get_material_details`, `calculate_quantity`

**CLIENT TOOLS** = Browser-side UI manipulation:
- Execute in user's browser via `commandStore` JavaScript
- No backend calls - commands pushed to client via polling
- Purpose: Cart updates, navigation, form filling, localStorage
- Tools: `create_or_update_cart`, `navigate_to`, `prefill_checkout_form`, `get_session_state`, `update_session_state`

**CRITICAL**: These tool types are NOT interchangeable. Data operations must be webhooks. UI manipulation must be client tools.

#### ElevenLabs Webhook Tool JSON Format

```json
{
  "type": "webhook",
  "name": "check_service_area",
  "description": "Checks if ZIP code is in service area",
  "api_schema": {
    "url": "https://milestone-trucks-chat.vercel.app/api/elevenlabs/webhook",
    "method": "POST",
    "content_type": "application/json",
    "request_body_schema": {
      "type": "object",
      "properties": {
        "tool_name": {
          "type": "string",
          "constant_value": "check_service_area"
        },
        "zip_code": {
          "type": "string",
          "description": "5-digit ZIP code"
        }
      },
      "required": ["tool_name", "zip_code"]
    }
  },
  "response_timeout_secs": 10,
  "execution_mode": "immediate"
}
```

**IMPORTANT**: No outer `tool_config` wrapper. Flat structure with `type` at root level.

#### Development Commands

```bash
cd " Milestone Voice Agent/milestone-trucks-chat"
npm run dev          # Start dev server (localhost:3000)
npm run build        # Build for production
npm run start        # Start production server
npm run lint         # Run ESLint
npm run create-tools # Generate ElevenLabs tool configs
```

#### Key Files

- `app/api/elevenlabs/webhook/route.ts` - Main webhook handler (all 4 webhook tools + client command queue)
- `lib/service-area-zips.ts` - 7,802 ZIP codes database
- `lib/materials.ts` - 14 material definitions (Product IDs 101-114)
- `lib/woocommerce.ts` - WooCommerce API client
- `lib/commandStore.ts` - Client-side command queue
- `ROBERT_SYSTEM_PROMPT.md` - Complete agent personality
- `project_requirements.md` - Technical constraints

### 2. milestone-voice-agent (Vercel Functions - Standalone)

**Location 1**: ` Milestone Voice Agent/milestone-voice-agent/`
**Location 2**: `Milestone Products /-milestone-voice-agent/` (GitHub clone)
**GitHub**: https://github.com/max-strong-1/-milestone-voice-agent
**Platform**: Vercel Serverless Functions (Node.js)
**Status**: Alternative implementation

#### Architecture

Individual API endpoints instead of unified webhook:
- `/api/check-service-area.js` - ZIP validation
- `/api/get-material-recommendations.js` - Project-based recommendations
- `/api/calculate-materials.js` - Quantity calculations
- `/api/calculate-delivery.js` - Delivery fee estimation
- `/api/add-to-cart.js` - Cart creation
- `/api/prefill-checkout.js` - Checkout preparation
- `/api/check-order-status.js` - Order lookup
- `/api/health.js` - Health check endpoint
- `/api/clear-cache.js` - Cache management

#### Development Commands

```bash
cd " Milestone Voice Agent/milestone-voice-agent"
npm run dev          # Start Vercel dev server
npm run deploy       # Deploy to production
npm run test         # Test endpoints
```

#### Key Libraries

- `lib/woocommerce.js` - WooCommerce REST API wrapper (10.5KB)
- `lib/calculations.js` - Material quantity formulas (7.3KB)
- `lib/materials-knowledge.js` - Educational content database (13KB)
- `lib/logger.js` - Structured logging utility (4.6KB)

#### Known Issues (from PROJECT_REVIEW.md)

✅ **Completed**:
- Cart storage not persistent
- Checkout prefill doesn't actually prefill

### 3. elevenlabs-woocommerce-voice-agent (Flask/Python)

**Location**: `Milestone Products /elevenlabs-woocommerce-voice-agent/`
**GitHub**: https://github.com/max-strong-1/elevenlabs-woocommerce-voice-agent
**Platform**: Python 3.9+ Flask
**Status**: Alternative Python implementation

#### Architecture

Flask webhook server with command queue system:
```
Customer Voice → ElevenLabs Widget → ElevenLabs AI
    → Webhook Server (Flask) → WooCommerce API
    → Client Commands (JavaScript polling)
```

#### Key Components

- `server/app.py` - Main Flask webhook handler (11KB)
- `server/woocommerce_client.py` - WooCommerce API wrapper (6.9KB)
- `server/zip_validator.py` - Zip → region mapping (2.4KB)
- `server/product_consultant.py` - AI recommendations (5.4KB)
- `server/command_queue.py` - Client command management (3KB)
- `config/zip_regions.json` - Zip code database

#### Development Commands

```bash
cd "Milestone Products /elevenlabs-woocommerce-voice-agent"
pip install -r server/requirements.txt
python server/app.py  # Start Flask dev server
```

#### Deployment

Recommended: Railway, Render, or Vercel
Requires: `WOOCOMMERCE_URL`, `WC_CONSUMER_KEY`, `WC_CONSUMER_SECRET`

#### ElevenLabs Tools (6 JSON configs)

**Webhook Tools**:
- `set_delivery_location.json` - Validate ZIP
- `consult_material.json` - Recommend products
- `search_products.json` - Query catalog
- `get_product_details.json` - Fetch specs

**Client Tools**:
- `add_to_cart.json` - Add items to cart
- `navigate_to_checkout.json` - Navigate to checkout page

## Data Sources

### Service Area ZIPs

- **Total**: 7,802 ZIP codes
- **States**: Ohio, Indiana, Pennsylvania, West Virginia, Kentucky, Michigan
- **Source**: `lib/service-area-zips.ts` (milestone-trucks-chat)
- **Function**: `checkServiceArea(zipCode)` returns boolean

### Materials Catalog

**14 Materials** with Product IDs 101-114:
- 101: #304 Limestone Base
- 102: #57 Limestone/Gravel
- 103: #2 Stone
- 104: #8 Gravel
- 105: River Gravel
- 106: Concrete Sand
- 107: Mulch (Black)
- 108: Topsoil (Screened)
- 109: Pea Gravel
- 110: Fill Dirt
- 111: #411 Limestone
- 112: Mulch (Brown)
- 113: Mulch (Red)
- 114: Mason Sand

**Source of Truth**: WooCommerce product database
**Delivery Fee**: $75 flat rate
**Pricing**: Variable by material (fetched from WooCommerce)

### Knowledge Base

**Location**: `Milestone Rag (Knowledge Base)/`

**Structure**:
- `01_company/` (4 files) - About, brand voice, hours, territory
- `02_materials/` (14 files) - Material definitions matching Product IDs
- `03_project_guides/` (6 files) - Project-specific guidance
- `04_calculations/` (4 files) - Quantity formulas
- `05_delivery/` (6 files) - Delivery logistics
- `06_customer_education/` (4 files) - Educational content
- `07_objections/` (5 files) - Objection handling scripts
- `08_safety/` (4 files) - Safety warnings
- `09_fallbacks/` (5 files) - Error handling scripts

**Purpose**: Training content for voice agent, RAG retrieval, customer education

## Robert Voice Agent Personality

**Tone**: Blue-collar foreman - patient, down-to-earth, helpful neighbor
**Style**: Casual contractions, celebrates questions, checks understanding

### Pronunciation Rules

**Critical for text-to-speech**:
- `#57` → "fifty-seven" (NOT "number fifty-seven")
- `#304` → "three-oh-four" (NOT "three hundred four")
- `#411` → "four-one-one" (NOT "four hundred eleven")
- `$392` → "three ninety-two"
- Quantities → Round them: "about fifteen tons" not "14.7 tons"

### Conversation Flow (Mandatory Order)

1. **Get ZIP code** (ALWAYS FIRST) - Call `check_service_area`
2. **Check service area** - Validate delivery availability
3. **Determine project type** - Driveway, walkway, patio, drainage, landscaping
4. **Recommend materials** - Based on project + availability
5. **Calculate quantities** - Get dimensions, call `calculate_quantity`
6. **Add to cart** - Customer confirmation required
7. **Collect customer info** - Name, email, phone, address
8. **Navigate to checkout** - Final step

### Quantity Calculation Formula

```javascript
// Depth in inches, dimensions in feet
cubicYards = (length × width × (depth/12)) ÷ 27
roundedYards = Math.ceil(cubicYards * 10) / 10  // Round to 1 decimal
```

**Rule of thumb**: ~1 ton per 10 square feet at 4 inches depth

## Product Management System

**Location**: `Milestone Products /`

### Files

- `Alliance MASTER CSV.csv` (301KB) - Master product database
- `Bulk CSV Product Update with API Integration and Error Tracking.json` - n8n workflow
- `Milestone Product Updater – SOP Pipeline.json` - n8n SOP workflow
- Product exports from WooCommerce (various formats)

### Tools

- CSV processing for bulk updates
- n8n workflows for automated product synchronization
- WooCommerce product exports (PDF, XML, CSV)

## Environment Variables

### All Voice Agent Implementations

```bash
WOOCOMMERCE_URL=https://staging12.milestonetrucks.com
WOOCOMMERCE_CONSUMER_KEY=ck_xxxxxxxxxxxxx
WOOCOMMERCE_CONSUMER_SECRET=cs_xxxxxxxxxxxxx
```

### Get WooCommerce API Keys

1. **WooCommerce → Settings → Advanced → REST API**
2. Click "Add Key"
3. Description: "Voice Agent API"
4. Permissions: Read/Write
5. Generate and copy Consumer Key + Secret

## Common Development Tasks

### Add a New Material

1. Add product to WooCommerce with Product ID 101-114
2. Create markdown file in `Milestone Rag (Knowledge Base)/02_materials/`
3. Update `lib/materials.ts` if using hardcoded definitions
4. Voice agent automatically fetches via `get_materials_by_zip`

### Add a New Webhook Tool (milestone-trucks-chat)

1. Determine if webhook (data) or client (UI)
2. Add case to `app/api/elevenlabs/webhook/route.ts` switch statement
3. If client tool, add command type to `lib/commandStore.ts`
4. Create JSON configuration (see format above)
5. Test endpoint locally first
6. Add tool in ElevenLabs dashboard

### Add a New ZIP Code

1. Edit `lib/service-area-zips.ts`
2. Add ZIP to `SERVICED_ZIP_CODES` array
3. Rebuild and redeploy
4. ZIP automatically validated by `checkServiceArea()`

### Update Agent Personality

Edit `ROBERT_SYSTEM_PROMPT.md`, then update in ElevenLabs dashboard under agent configuration.

### Debug Tool Execution

**Webhook tools**:
1. Check Vercel function logs
2. Test with curl: `curl -X POST https://milestone-trucks-chat.vercel.app/api/elevenlabs/webhook -H "Content-Type: application/json" -d '{"tool_name": "check_service_area", "zip_code": "43560"}'`
3. Verify `tool_name` matches switch case exactly

**Client tools**:
1. Check browser console for command execution
2. Verify frontend is polling `/api/elevenlabs/commands`
3. Check command queue in `commandStore`

## Testing

### Test Service Area Validation

```bash
curl -X POST https://milestone-trucks-chat.vercel.app/api/elevenlabs/webhook \
  -H "Content-Type: application/json" \
  -d '{"tool_name": "check_service_area", "zip_code": "43560"}'
```

Expected: `{"result": "Yes, we service ZIP code 43560!..."}`

### Test Material Calculation

```bash
curl -X POST https://milestone-trucks-chat.vercel.app/api/elevenlabs/webhook \
  -H "Content-Type: application/json" \
  -d '{"tool_name": "calculate_quantity", "length": 20, "width": 30, "depth": 4}'
```

Expected: `{"cubic_yards": 7.4, "message": "For an area 20' × 30' with 4\" depth..."}`

### End-to-End Voice Flow Test

1. Say: "Hi Robert, I'm in ZIP 43560"
2. Verify `check_service_area` called
3. Say: "I'm building a driveway"
4. Verify `get_materials_by_zip` called
5. Say: "It's 50 feet long and 12 feet wide, 4 inches deep"
6. Verify `calculate_quantity` called
7. Complete cart/checkout flow

## Common Issues and Solutions

### Error: Invalid discriminator value (ElevenLabs)

**Problem**: Incorrect JSON structure or nesting
**Solution**: Remove outer `tool_config` wrapper, use flat structure with `"type": "webhook"` at root

### Error: Tool not executing

**Problem**: Tool configured as CLIENT when should be WEBHOOK (or vice versa)
**Solution**: Verify tool type matches function - data operations = webhook, UI manipulation = client

### High LLM processing time (13-17s)

**Status**: Normal behavior, not a bug
**Breakdown**:
- LLM Processing: ~17s
- Text-to-Speech: ~225ms
- RAG Retrieval: ~344ms
- Tool Execution: 1-259ms (client) or varies (webhook)

### Products not found for ZIP

**Problem**: ZIP not in service area database
**Solution**: Add ZIP to `lib/service-area-zips.ts`

### Cart not updating (client tools)

**Problem**: Frontend not polling commands endpoint
**Solution**: Verify frontend polls `/api/elevenlabs/commands` with correct `call_id`

## Integration Points

### WooCommerce Requirements

Products must have:
1. **Tags**: Region tags (e.g., `region_wooster_oh`) for ZIP filtering
2. **Custom Fields**: `density`, `truck_max_quantity`, `minimum_quantity`, `map_title`
3. **Pricing**: Regular price set in WooCommerce
4. **Stock Status**: "In Stock" for available products

### Never Expose WooCommerce API Directly

Always use Next.js/Vercel backend as middleman. ElevenLabs should NEVER call WooCommerce directly.

### ZIP Validation is Mandatory First Step

Agent MUST call `check_service_area` before recommending materials or discussing pricing.

## Architecture Documentation

- **Tool Architecture**: `Milestone Products /Milestone Trucks Voice Agent - Tool Architecture Documentation.md`
- **ElevenLabs Setup**: ` Milestone Voice Agent/milestone-voice-agent/docs/ELEVENLABS_SETUP.md`
- **System Prompt**: ` Milestone Voice Agent/milestone-trucks-chat/ROBERT_SYSTEM_PROMPT.md`
- **Project Requirements**: ` Milestone Voice Agent/milestone-trucks-chat/project_requirements.md`
- **Project Review**: `Milestone Products /-milestone-voice-agent/docs/PROJECT_REVIEW.md`
- **Latency Optimization**: `Milestone Products /-milestone-voice-agent/docs/LATENCY_OPTIMIZATION.md`

## Performance Metrics

From production testing:
- LLM Processing: ~17s
- Text-to-Speech: ~225ms
- RAG Retrieval: ~344ms
- Webhook Tool Execution: Varies by complexity
- Client Tool Execution: ~1-259ms
- Pre-fill form execution: 259ms request, 1ms result
- Cart update: 239ms request, 1ms result

## Repository Navigation

Due to spaces in directory names, always quote paths:

```bash
cd " Milestone Voice Agent/milestone-trucks-chat"
cd "Milestone Products /elevenlabs-woocommerce-voice-agent"
cd "Milestone Rag (Knowledge Base)/02_materials"
```

## Contact Information

- **Developer**: Kel Mewbourne
- **Company**: Foreman AI
- **Email**: maxstrongperformance@gmail.com
- **Phone**: 561-831-9310
