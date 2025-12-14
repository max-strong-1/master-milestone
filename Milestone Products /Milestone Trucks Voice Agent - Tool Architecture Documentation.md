# **Milestone Trucks Voice Agent \- Tool Architecture Documentation**

## **Overview**

This document captures the critical architectural decisions and configurations for Robert, the Milestone Trucks voice agent built on ElevenLabs.

## **Repository & Deployment**

* GitHub: github.com/max-strong-1/milestone-trucks-chat  
* Live Site: milestone-trucks-chat.vercel.app  
* Platform: ElevenLabs (NOT Retell \- important distinction)  
* Backend: Next.js deployed on Vercel

## **Service Scope**

* Service Area: 7,802 ZIP codes across OH, IN, PA, WV, MI  
* Product Catalog: 14 materials (Product IDs 101-114)  
* Delivery Fee: $75 flat rate  
* Minimum Quantities: Vary by material

---

## **Critical Architecture: Webhook vs Client Tools**

### **THE KEY DISTINCTION**

WEBHOOK TOOLS \= Server-side data operations (hit Vercel backend)

* Used for: Database queries, calculations, validation logic  
* Call endpoint: https://milestone-trucks-chat.vercel.app/api/retell/webhook  
* Processed by: /app/api/retell/webhook/route.ts

CLIENT TOOLS \= Browser-side UI manipulation (run in user's browser)

* Used for: Cart updates, navigation, form filling, localStorage  
* Executed via: commandStore JavaScript in browser  
* No backend calls involved

---

## **Tool Inventory**

### **Webhook Tools (4 total)**

These call the Next.js backend:

1. check\_service\_area  
   * Validates ZIP against 7,802 ZIP database  
   * Data source: /lib/service-area-zips.ts  
2. get\_materials\_by\_zip  
   * Fetches available materials for a ZIP code  
   * Data source: WooCommerce API via Next.js middleware  
3. get\_material\_details  
   * Returns product specifications, pricing, minimums  
   * Data source: WooCommerce product catalog (IDs 101-114)  
4. calculate\_quantity  
   * Cubic yard calculations (Length × Width × Depth ÷ 27\)  
   * Pure calculation logic on backend

### **Client Tools (5 total)**

These manipulate the website in real-time:

1. create\_or\_update\_cart  
   * Add/update WooCommerce cart items  
   * Sends commands to browser JavaScript  
2. navigate\_to  
   * Change pages (/checkout, /products, /contact)  
   * Client-side routing  
3. prefill\_checkout\_form  
   * Auto-fill customer data into form fields  
   * Direct DOM manipulation  
4. get\_session\_state  
   * Read browser localStorage  
   * Client-side data retrieval  
5. update\_session\_state  
   * Write to browser localStorage  
   * Client-side data storage

---

## **Webhook Tool Configuration**

### **JSON Structure for ElevenLabs**

All webhook tools use this format (NO outer tool\_config wrapper):  
{  
  "type": "webhook",  
  "name": "check\_service\_area",  
  "description": "Checks if a ZIP code is within Milestone Trucks' service area covering Ohio, Indiana, Pennsylvania, West Virginia, and Michigan. Use this before quoting materials or taking orders.",  
  "api\_schema": {  
    "url": "https://milestone-trucks-chat.vercel.app/api/retell/webhook",  
    "method": "POST",  
    "content\_type": "application/json",  
    "request\_body\_schema": {  
      "type": "object",  
      "properties": {  
        "tool\_name": {  
          "type": "string",  
          "constant\_value": "check\_service\_area"  
        },  
        "zip\_code": {  
          "type": "string",  
          "description": "5-digit ZIP code to check"  
        }  
      },  
      "required": \["tool\_name", "zip\_code"\]  
    }  
  },  
  "response\_timeout\_secs": 10,  
  "execution\_mode": "immediate"  
}

### **Backend Response Format**

Expected from /api/retell/webhook/route.ts:  
case "check\_service\_area": {  
  const { zip\_code } \= tool\_call\_data;  
  const isServiced \= checkServiceArea(zip\_code);  
  return NextResponse.json({  
    result: isServiced   
      ? \`Yes, we service ZIP code ${zip\_code}\! It's within our delivery area.\`  
      : \`Sorry, ZIP code ${zip\_code} is outside our current service area.\`  
  });  
}

---

## **Data Sources**

### **Service Area ZIPs**

* Location: /lib/service-area-zips.ts  
* Count: 7,802 ZIP codes  
* States: Ohio, Indiana, Pennsylvania, West Virginia, Michigan

### **Materials Catalog**

* Source of Truth: WooCommerce product database  
* Product IDs: 101-114  
* API Endpoint: /app/api/materials/route.ts  
* Pricing: Variable by product  
* Delivery: $75 flat fee

### **Architecture Decision**

Use Next.js backend as middleman for ALL server data. No direct WooCommerce API calls from ElevenLabs.  
---

## **Observed Performance Metrics**

### **Tool Execution Times (from test conversation)**

* prefill\_checkout\_form: 259ms request, 1ms result  
* create\_or\_update\_cart: 239ms request, 1ms result  
* Second cart call: 13s LLM processing time

### **Latency Breakdown**

* LLM Processing: \~17s  
* Text-to-Speech: \~225ms  
* RAG Retrieval: \~344ms

---

## **Conversation Flow Validation**

Robert successfully demonstrated:

* ✅ ZIP code collection (43560)  
* ✅ Customer qualification (homeowner)  
* ✅ Project understanding (concrete pad, driveway)  
* ✅ Order taking (3 tons \#57 limestone \+ 5cy \#304 limestone)  
* ✅ Automatic quantity calculations  
* ✅ Contact info collection (name, email, phone, address)  
* ✅ Checkout form pre-filling  
* ✅ Cart item addition  
* ✅ Navigation to checkout

### **Conversational Quality**

Robert exhibits:

* Natural thinking pauses ("Hhmmmm...yeah give me a second")  
* Helpful confirmations  
* Friendly, professional tone  
* No robotic patterns

---

## **Common Validation Errors & Solutions**

### **Error: Invalid discriminator value**

Problem: Incorrect JSON structure or nesting  
Solution: Remove outer tool\_config wrapper, use flat structure with "type": "webhook" at root level

### **Error: Tool not executing**

Problem: Tool configured as CLIENT when it should be WEBHOOK (or vice versa)  
Solution: Verify tool type matches its function (data \= webhook, UI \= client)  
---

## **Migration Checklist**

When converting CLIENT tools to WEBHOOK tools:

*  Create new WEBHOOK tool in ElevenLabs with correct JSON  
*  Test webhook endpoint responds correctly  
*  Verify tool appears in agent's available tools  
*  Test tool execution in conversation  
*  Delete old CLIENT version to avoid conflicts  
*  Update agent prompt if tool usage changed

---

## **Next Steps**

1. ✅ Create check\_service\_area webhook tool  
2. ⏳ Create calculate\_quantity webhook tool  
3. ⏳ Create get\_materials\_by\_zip webhook tool  
4. ⏳ Create get\_material\_details webhook tool  
5. ⏳ Delete old CLIENT versions of these 4 tools  
6. ⏳ Verify /api/retell/webhook/route.ts handles all 4 cases  
7. ⏳ End-to-end test with Robert

---

## **Key Learnings**

1. Tool type matters: Webhook vs Client is not interchangeable  
2. Data belongs on server: Validation, calculations, database queries should use webhooks  
3. UI belongs in browser: Cart, navigation, forms should use client tools  
4. JSON structure is strict: No extra wrappers, flat structure with type at root  
5. Backend is middleman: Never expose WooCommerce API directly to ElevenLabs  
6. Test incrementally: Create one tool, test, then move to next

---

## **Contact & Support**

* Developer: Kel Mewbourne  
* Company: Foreman AI  
* Email: maxstrongperformance@gmail.com  
* Phone: 561-831-9310

---

*Last Updated: December 2024* *Agent Version: Robert v1.0* *Platform: ElevenLabs Conversational AI*  
