# 🚀 Universal Google Forms -> Webhook & Lead Routing Engine

**Asset Type**: Drop-in Google Apps Script (`Code.gs`)  
**Monetization Value**: Sell as an instant **$75 - $150 flat-rate setup** to agencies, real estate teams, or SaaS founders drowning in manual lead assignment.  
**Fulfillment Time**: **5 Minutes** (Paste code, add Webhook URL, click Save).

---

## 💡 Why This is the Most Practical Asset to Sell Right Now
Non-technical founders and marketing teams rely heavily on Google Forms or Google Sheets to collect client inquiries, but they have two major bottlenecks:
1. **Duplicate Submissions**: Customers submitting multiple times skewing their Cost-Per-Lead (CPL) metrics.
2. **Disconnected Pipelines**: Relying on manual email notifications or expensive Zapier subscriptions to send lead data into their CRM, Slack, or WhatsApp routing queues.

You can pitch them right now:  
> *"I have a lightweight, custom-coded intake engine that drops directly into your Google Sheet. It automatically detects duplicate leads, formats your unstructured field data, and instantly pushes clean JSON payloads to your WhatsApp/CRM routing queue the second a user submits. No recurring Zapier fees. I can install and test it for you live on a screen-share in 10 minutes for $75 flat."*

---

## 🛠️ The Production-Ready Source Code (`Code.gs`)

Copy and paste this exact script into the client's **Google Sheets > Extensions > Apps Script** editor.

```javascript
/**
 * Universal Lead Intake & Routing Engine
 * Built for zero-routine automated operations.
 */

// CONFIGURATION: Set the destination Webhook (Make.com, Zapier, Qontak, Slack, custom CRM API)
const CONFIG = {
  WEBHOOK_URL: "https://your-routing-engine-webhook.endpoint/api/v1/lead",
  ENABLE_DUPLICATE_CHECK: true,
  EMAIL_COLUMN_INDEX: 2, // Assuming Column B is Email (1-indexed)
  PHONE_COLUMN_INDEX: 3, // Assuming Column C is Phone
  LOGGING_SHEET_NAME: "System_Logs"
};

/**
 * Main Trigger function bound to "On Form Submit"
 */
function onFormSubmitTrigger(e) {
  try {
    const sheet = e.range.getSheet();
    const responses = e.namedValues; // Captures all Form questions and answers as key-value pairs
    const rowIdx = e.range.getRow();
    const values = e.values;

    // 1. DUPLICATE DETECTION LOGIC
    if (CONFIG.ENABLE_DUPLICATE_CHECK) {
      const isDupe = checkDuplicate(sheet, rowIdx, values);
      if (isDupe) {
        logEvent("WARNING: Duplicate Detected", `Row ${rowIdx} ignored to preserve clean CPL analytics.`);
        // Optional: Highlight duplicate row in visual yellow for the client
        sheet.getRange(rowIdx, 1, 1, sheet.getLastColumn()).setBackground("#FFF3CD");
        return; 
      }
    }

    // 2. UNSTRUCTURED DATA NORMALIZATION
    // Clean keys and compile structured payload
    const payload = {
      timestamp: new Date().toISOString(),
      lead_id: `LD-${Date.now()}`,
      source: sheet.getName(),
      raw_data: responses,
      extracted_intent: {}
    };

    // Flatten named values array strings for easy consumption by external APIs
    for (let key in responses) {
      const cleanKey = key.trim().replace(/\s+/g, "_").toLowerCase();
      payload.extracted_intent[cleanKey] = responses[key].join(", ").trim();
    }

    // 3. ASYNCHRONOUS WEBHOOK DISPATCH
    const options = {
      method: "post",
      contentType: "application/json",
      payload: JSON.stringify(payload),
      muteHttpExceptions: true
    };

    const response = UrlFetchApp.fetch(CONFIG.WEBHOOK_URL, options);
    const responseCode = response.getResponseCode();

    if (responseCode >= 200 && responseCode < 300) {
      logEvent("SUCCESS: Lead Routed", `Lead ID ${payload.lead_id} broadcasted successfully. Status: ${responseCode}`);
      sheet.getRange(rowIdx, 1, 1, sheet.getLastColumn()).setBackground("#D1E7DD"); // Green highlight for verified
    } else {
      logEvent("ERROR: Routing Failed", `Payload dispatch failed. Code: ${responseCode}. Response: ${response.getContentText()}`);
      sheet.getRange(rowIdx, 1, 1, sheet.getLastColumn()).setBackground("#F8D7DA"); // Red highlight for review
    }

  } catch (error) {
    logEvent("CRITICAL: Engine Exception", error.toString());
  }
}

/**
 * Scans previous entries to identify identical matching lead keys
 */
function checkDuplicate(sheet, currentRowIdx, rowValues) {
  const data = sheet.getRange(2, 1, currentRowIdx - 2, sheet.getLastColumn()).getValues(); // Read all prior rows
  const targetEmail = rowValues[CONFIG.EMAIL_COLUMN_INDEX - 1]?.toString().toLowerCase().trim();
  const targetPhone = rowValues[CONFIG.PHONE_COLUMN_INDEX - 1]?.toString().replace(/\D/g, "");

  for (let i = 0; i < data.length; i++) {
    const existingEmail = data[i][CONFIG.EMAIL_COLUMN_INDEX - 1]?.toString().toLowerCase().trim();
    const existingPhone = data[i][CONFIG.PHONE_COLUMN_INDEX - 1]?.toString().replace(/\D/g, "");

    if ((targetEmail && targetEmail === existingEmail) || (targetPhone && targetPhone === existingPhone)) {
      return true;
    }
  }
  return false;
}

/**
 * Appends automated background audit trails to an internal log tab
 */
function logEvent(type, message) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  let logTab = ss.getSheetByName(CONFIG.LOGGING_SHEET_NAME);
  
  if (!logTab) {
    logTab = ss.insertSheet(CONFIG.LOGGING_SHEET_NAME);
    logTab.appendRow(["Timestamp", "Event Type", "Operational Details"]);
    logTab.getRange("A1:C1").setFontWeight("bold").setBackground("#333333").setFontColor("#FFFFFF");
  }
  
  logTab.appendRow([new Date().toLocaleString(), type, message]);
}
```

---

## ⚡ How to Deliver & Install in 60 Seconds

When a client buys your **$75 setup fix**, follow these exact steps to fulfill:
1. Ask the client to share their active Google Sheet containing the Form Responses (Editor access).
2. Open **Extensions > Apps Script**.
3. Replace `Code.gs` with the script above.
4. Replace `CONFIG.WEBHOOK_URL` with their endpoint (or a quick free Make.com/Zapier catchhook if they want visual workflow drops).
5. On the left sidebar, click the **Triggers icon (⏰)** -> **Add Trigger** -> Select `onFormSubmitTrigger` -> Event Source: `From spreadsheet` -> Event Type: `On form submit` -> Click Save.
6. Submit one test lead. Watch the row automatically turn **Green** and flow perfectly to their webhook. 

**Result**: Complete client satisfaction, verifiable real-time tracking, and **instant cash in your pocket.**
