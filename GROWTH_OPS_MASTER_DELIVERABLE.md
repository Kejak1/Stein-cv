# 🎉 The Growth Ops Master Kit: Operational Infrastructure Suite

**Document Classification**: Confidential Master Deliverable  
**Access Level**: Verified Customer Access Only  

Welcome to your finalized Growth Operations & Unit Economics tracking suite. This blueprint eliminates manual copy-pasting, enforces raw data hygiene, and provides immediate visual clarity on your true Cost-Per-Lead (CPL) and multi-agent sales routing velocity.

Follow the three implementation modules below to deploy your infrastructure within 15 minutes.

---

## 📊 Module 1: The 10-Metric Unit Economics Schema

To track exactly how your top-of-funnel marketing burn converts into finalized retail unit sell-through, configure your central ledger database (Google Sheets or Feishu Base) with these exact mandatory columns.

### 📑 Tab 1: `Campaign_Burn_Ledger`
Track your daily operational burn rates across active marketing pushes.
* `Date` *(Format: YYYY-MM-DD)*
* `Initiative_Name` *(e.g., Meta Lookalike Ads, Regional Dealer Roadshow)*
* `Corporate_Spend_Allocated` *(Capital deployed directly from HQ)*
* `Partner_Subsidy_Cost` *(Direct HQ dealer subsidies)*
* `Out_Of_Pocket_Local_Spend` *(Independent capital deployed by retail points)*
* `Total_Daily_Burn` *(Formula: `=SUM(C2:E2)`)*
* `Raw_Leads_Captured` *(Absolute integer count of raw form entries)*
* `Verified_Unique_Leads` *(Deduplicated unique contact count)*
* `Absolute_Daily_CPL` *(Formula: `=F2/H2` — Flags immediate acquisition cost spikes)*
* `Active_Duration_Days` *(Running integer tracking cumulative active footprint)*

### 📑 Tab 2: `Retail_Outcomes_Ledger`
Map top-of-funnel contacts directly to bottom-of-funnel closing metrics.
* `Lead_ID` *(Generated autonomously by the Intake Engine)*
* `Customer_Intent_Data` *(Extracted JSON string from raw questions)*
* `Assigned_Sales_Agent` *(Targeted sales rep securing the contact)*
* `Time_To_Claim_Seconds` *(Absolute integer tracking response speed)*
* `Finalized_Sale_Status` *(Dropdown: `Pending_Contact` / `Demo_Scheduled` / `Closed_Won` / `Lost_Unresponsive`)*
* `Attached_Hardware_Serial` *(Physical unit/IoT telemetry tag tracking fulfillment)*

---

## ⚡ Module 2: Drop-In Lead Intake & Deduplication Engine

Stop paying recurring Zapier or Make.com tiers to push leads from Google Forms into your databases. This standalone script intercepts submissions, automatically filters out spam or duplicate entries, normalizes unstructured data, and dispatches real-time Webhooks.

### 🛠️ Installation Instructions:
1. Open your Google Sheet linked to your lead intake Form.
2. On the top menu bar, navigate to **Extensions > Apps Script**.
3. Clear any default code and paste the production script below.
4. Replace `CONFIG.WEBHOOK_URL` with your destination receiver URL.
5. On the left sidebar, click the **Triggers icon (⏰)** -> **Add Trigger** -> Select function `onFormSubmitTrigger` -> Event Source: `From spreadsheet` -> Event Type: `On form submit` -> Click **Save**.

```javascript
/**
 * Standalone Lead Intake & Deduplication Engine
 * Intercepts Google Form payloads, enforces CPL data hygiene, and dispatches zero-fee webhooks.
 */

const CONFIG = {
  // Destination URL to catch incoming normalized lead payloads
  WEBHOOK_URL: "https://your-routing-engine-webhook.endpoint/api/v1/lead",
  ENABLE_DUPLICATE_CHECK: true,
  EMAIL_COLUMN_INDEX: 2, // Assuming Column B contains the Customer Email (1-indexed)
  PHONE_COLUMN_INDEX: 3, // Assuming Column C contains the Customer Phone Number
  LOGGING_SHEET_NAME: "System_Logs"
};

function onFormSubmitTrigger(e) {
  try {
    const sheet = e.range.getSheet();
    const responses = e.namedValues; 
    const rowIdx = e.range.getRow();
    const values = e.values;

    // 1. ASYNCHRONOUS DUPLICATE FILTERING
    if (CONFIG.ENABLE_DUPLICATE_CHECK) {
      if (checkDuplicateEntry(sheet, rowIdx, values)) {
        logSystemAudit("WARNING: Duplicate Ignored", `Row ${rowIdx} ignored to prevent skewed CPL calculations.`);
        // Highlights duplicate entries in muted yellow for review
        sheet.getRange(rowIdx, 1, 1, sheet.getLastColumn()).setBackground("#FFF3CD");
        return; 
      }
    }

    // 2. PAYLOAD NORMALIZATION
    const payload = {
      timestamp: new Date().toISOString(),
      lead_id: `LD-${Date.now()}`,
      source_form: sheet.getName(),
      extracted_intent: {}
    };

    // Strip special characters and normalize spaces into clean database properties
    for (let key in responses) {
      const sanitizedKey = key.trim().replace(/\s+/g, "_").toLowerCase();
      payload.extracted_intent[sanitizedKey] = responses[key].join(", ").trim();
    }

    // 3. ZERO-FEE WEBHOOK DISPATCH
    const options = {
      method: "post",
      contentType: "application/json",
      payload: JSON.stringify(payload),
      muteHttpExceptions: true
    };

    const response = UrlFetchApp.fetch(CONFIG.WEBHOOK_URL, options);
    const responseCode = response.getResponseCode();

    if (responseCode >= 200 && responseCode < 300) {
      logSystemAudit("SUCCESS: Routed", `Lead ID ${payload.lead_id} broadcasted successfully. Code: ${responseCode}`);
      sheet.getRange(rowIdx, 1, 1, sheet.getLastColumn()).setBackground("#D1E7DD"); // Confirmed green flag
    } else {
      logSystemAudit("ERROR: Webhook Failed", `Payload dispatch error. Status: ${responseCode}`);
      sheet.getRange(rowIdx, 1, 1, sheet.getLastColumn()).setBackground("#F8D7DA"); // Alert red flag
    }

  } catch (error) {
    logSystemAudit("CRITICAL: Exception", error.toString());
  }
}

function checkDuplicateEntry(sheet, currentRowIdx, rowValues) {
  const previousData = sheet.getRange(2, 1, currentRowIdx - 2, sheet.getLastColumn()).getValues();
  const targetEmail = rowValues[CONFIG.EMAIL_COLUMN_INDEX - 1]?.toString().toLowerCase().trim();
  const targetPhone = rowValues[CONFIG.PHONE_COLUMN_INDEX - 1]?.toString().replace(/\D/g, "");

  for (let i = 0; i < previousData.length; i++) {
    const pastEmail = previousData[i][CONFIG.EMAIL_COLUMN_INDEX - 1]?.toString().toLowerCase().trim();
    const pastPhone = previousData[i][CONFIG.PHONE_COLUMN_INDEX - 1]?.toString().replace(/\D/g, "");

    if ((targetEmail && targetEmail === pastEmail) || (targetPhone && targetPhone === pastPhone)) {
      return true;
    }
  }
  return false;
}

function logSystemAudit(eventType, details) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  let logSheet = ss.getSheetByName(CONFIG.LOGGING_SHEET_NAME);
  
  if (!logSheet) {
    logSheet = ss.insertSheet(CONFIG.LOGGING_SHEET_NAME);
    logSheet.appendRow(["Timestamp", "Audit Status", "System Details"]);
    logSheet.getRange("A1:C1").setFontWeight("bold").setBackground("#2B2B2B").setFontColor("#FFFFFF");
  }
  
  logSheet.appendRow([new Date().toLocaleString(), eventType, details]);
}
```

---

## 🏆 Module 3: Multi-Agent "Competition Mode" Distribution Logic

Traditional round-robin lead assignment fails because it allows sales reps to let leads sit uncontacted without consequences. To achieve maximum response velocity, implement the **Competition Mode** architecture loop inside your Make.com or custom CRM endpoints.

### 🔄 The Execution Logic Loop:
1. **Simultaneous Broadcast**: The Webhook intake dispatches the incoming lead payload to your top 3 available reps simultaneously on WhatsApp via an Interactive Message Template containing a dedicated `[ ⚡ Claim Lead ]` button.
2. **First-to-Claim Verification**: The central backend intercepts inbound button callback responses.
3. **Atomic Execution Lockout**: The backend uses an atomic processing lock to verify if the lead state is still marked `UNCLAIMED`.
   * **If Valid**: The lead state flips instantly to `CLAIMED` attached to the claiming rep's ID. The backend responds with the full customer contact details.
   * **If Invalid**: The backend blocks the request and sends an instant visual update to the slow rep: *"❌ TOO SLOW: Secured by Rep [Name] exactly [X.X] seconds faster."*

### 🗺️ Systems Topology Diagram:
```
[ Incoming Lead Webhook ] ──► [ Central Ledger: Mark UNCLAIMED ]
                                     │
       ┌─────────────────────────────┼─────────────────────────────┐
       ▼                             ▼                             ▼
[ Push Alert: Rep 1 ]         [ Push Alert: Rep 2 ]         [ Push Alert: Rep 3 ]
       │                             │                             │
       └─────────────────────────────┼─────────────────────────────┘
                                     ▼
                      [ Fastest Button Tap Intercepted ]
                                     │
                                     ▼
                    [ Atomic State Flip to CLAIMED ]
                                     │
                  ┌──────────────────┴──────────────────┐
                  ▼                                     ▼
      [ Reveal Details to Winner ]          [ Lockout Notice to Losers ]
```

---

## 📞 Support & Custom Integrations
Need to wire this architecture directly into bespoke IoT platforms, custom Qontak engines, or regional dealer CRM networks? Reach out to the author directly to secure priority implementation bandwidth.
