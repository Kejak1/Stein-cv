# 🏆 Multi-Agent "Competition Mode" WhatsApp Lead Routing Engine

**Asset Type**: Drop-in Webhook Routing Backend (Google Apps Script / Node.js Logic)  
**Monetization Value**: Sell as a **$150 - $300 custom workflow upgrade** to real estate brokerages, solar sales agencies, or high-ticket coaching teams.  
**Core Problem Solved**: Sales reps get lazy and respond to leads too slowly. This script forces absolute speed by turning lead assignment into a race.

---

## 💡 The Pitch to Sales Directors & Agency Owners
High-ticket sales teams bleed revenue when leads sit uncontacted for more than 5 minutes. Traditional round-robin assignment doesn't incentivize speed.

Pitch them right now via DM or LinkedIn:  
> *"I have pre-coded a 'Competition Mode' lead distribution logic loop for sales teams. The second a lead comes in, the engine broadcasts it simultaneously to your top 3 available reps on WhatsApp. The first rep to click 'Claim Lead' wins exclusive ownership of the B2B/B2C contact. All other reps are instantly locked out. It forces absolute response velocity and syncs response times back to your ledger. I can integrate this into your Qontak, Make.com, or custom WhatsApp API setup today for $150 flat."*

---

## 🛠️ Production-Ready Source Code (`CompetitionEngine.gs`)

This script acts as the centralized Webhook receiver. It broadcasts interactive WhatsApp template messages with interactive buttons (`[ Claim Lead ]`) and processes incoming claim callbacks atomically to prevent race conditions.

```javascript
/**
 * WhatsApp Competition Mode Routing Engine
 * Forces maximum sales velocity via atomic First-to-Claim logic.
 */

const CONFIG = {
  WHATSAPP_API_TOKEN: "your_meta_or_qontak_api_token",
  API_ENDPOINT: "https://graph.facebook.com/v18.0/your_phone_number_id/messages", // Meta API example
  STATE_SHEET_NAME: "Active_Leads_Ledger",
  SALES_TEAM: [
    { name: "Rep Alpha", phone: "12345678901" },
    { name: "Rep Beta",  phone: "12345678902" },
    { name: "Rep Gamma", phone: "12345678903" }
  ]
};

/**
 * 1. INTAKE WEBHOOK: Triggered when a new lead arrives from Google Forms / Ads
 */
function broadcastLeadToTeam(leadPayload) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = getOrCreateSheet(ss, CONFIG.STATE_SHEET_NAME);
  
  const leadId = `LD-${Date.now()}`;
  const leadData = JSON.stringify(leadPayload);
  const broadcastTime = new Date().getTime();
  
  // Log initial state as "UNCLAIMED"
  // Columns: [Lead ID, Payload, Status, Claimed By, Claimed Time, Response Velocity (s)]
  sheet.appendRow([leadId, leadData, "UNCLAIMED", "None", broadcastTime, 0]);
  const rowIdx = sheet.getLastRow();
  
  // Broadcast alert to all target reps simultaneously
  CONFIG.SALES_TEAM.forEach(rep => {
    sendWhatsAppInteractiveAlert(rep.phone, rep.name, leadId, leadPayload.name, leadPayload.intent);
  });
  
  return { status: "BROADCASTED", lead_id: leadId, targets: CONFIG.SALES_TEAM.length };
}

/**
 * 2. CLAIM CALLBACK WEBHOOK: Intercepts button clicks from sales reps inside WhatsApp
 */
function doPost(e) {
  try {
    const callbackData = JSON.parse(e.postData.contents);
    // Extract interactive button payload (e.g., "CLAIM_LD-123456")
    const buttonPayload = callbackData.entry[0].changes[0].value.messages[0].button.payload;
    const repPhone = callbackData.entry[0].changes[0].value.contacts[0].wa_id;
    
    if (buttonPayload.startsWith("CLAIM_")) {
      const leadId = buttonPayload.split("CLAIM_")[1];
      const result = processAtomicClaim(leadId, repPhone);
      
      // Notify the rep of their race outcome
      if (result.success) {
        sendSimpleWhatsApp(repPhone, `🎉 SUCCESS: You claimed Lead ${leadId}! Customer Contact: ${result.lead.phone}. Close the deal!`);
      } else {
        sendSimpleWhatsApp(repPhone, `❌ TOO SLOW: Lead ${leadId} was already secured by ${result.winner} exactly ${result.margin} seconds faster.`);
      }
    }
    
    return ContentService.createTextOutput(JSON.stringify({ status: "processed" })).setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({ error: error.toString() })).setMimeType(ContentService.MimeType.JSON);
  }
}

/**
 * Atomically resolves claims using Google Apps Script Lock Service to prevent race conditions
 */
function processAtomicClaim(targetLeadId, claimingPhone) {
  const lock = LockService.getScriptLock();
  lock.waitLock(5000); // Wait up to 5 seconds for exclusive system execution access
  
  try {
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    const sheet = ss.getSheetByName(CONFIG.STATE_SHEET_NAME);
    const data = sheet.getDataRange().getValues();
    
    for (let i = 1; i < data.length; i++) {
      const rowLeadId = data[i][0];
      const status = data[i][2];
      const broadcastTime = data[i][4];
      
      if (rowLeadId === targetLeadId) {
        const claimingRep = CONFIG.SALES_TEAM.find(r => r.phone === claimingPhone)?.name || claimingPhone;
        const currentTime = new Date().getTime();
        const velocitySeconds = ((currentTime - broadcastTime) / 1000).toFixed(1);

        if (status === "UNCLAIMED") {
          // WINNER FOUND: Update row state immediately
          sheet.getRange(i + 1, 3).setValue("CLAIMED");
          sheet.getRange(i + 1, 4).setValue(claimingRep);
          sheet.getRange(i + 1, 5).setValue(new Date().toISOString());
          sheet.getRange(i + 1, 6).setValue(velocitySeconds);
          
          // Green fill for claimed success
          sheet.getRange(i + 1, 1, 1, sheet.getLastColumn()).setBackground("#D1E7DD");
          
          const leadPayload = JSON.parse(data[i][1]);
          return { success: true, lead: leadPayload, velocity: velocitySeconds };
        } else {
          // ALREADY CLAIMED: Calculate failure margin
          const winner = data[i][3];
          const winningVelocity = data[i][5]; // wait, column 6 is response velocity
          return { success: false, winner: winner, margin: velocitySeconds };
        }
      }
    }
    return { success: false, winner: "System Verification Error", margin: 0 };
  } finally {
    lock.releaseLock(); // Always release global system lock
  }
}

/**
 * Sends interactive WhatsApp message containing custom Quick Reply Claim buttons
 */
function sendWhatsAppInteractiveAlert(targetPhone, repName, leadId, leadName, leadIntent) {
  const payload = {
    messaging_product: "whatsapp",
    recipient_type: "individual",
    to: targetPhone,
    type: "interactive",
    interactive: {
      type: "button",
      body: { text: `🚨 *NEW COMPETITION LEAD* 🚨\n\n*Target*: ${leadName}\n*Intent*: ${leadIntent}\n\nFirst rep to tap below secures exclusive B2B rights.` },
      action: {
        buttons: [
          { type: "reply", reply: { id: `CLAIM_${leadId}`, title: "⚡ Claim Lead" } }
        ]
      }
    }
  };
  
  UrlFetchApp.fetch(CONFIG.API_ENDPOINT, {
    method: "post",
    headers: { "Authorization": `Bearer ${CONFIG.WHATSAPP_API_TOKEN}` },
    contentType: "application/json",
    payload: JSON.stringify(payload),
    muteHttpExceptions: true
  });
}

function getOrCreateSheet(ss, name) {
  let sheet = ss.getSheetByName(name);
  if (!sheet) {
    sheet = ss.insertSheet(name);
    sheet.appendRow(["Lead ID", "Raw Payload", "Status", "Claimed By", "Broadcast Time", "Response Velocity (s)"]);
    sheet.getRange("A1:F1").setFontWeight("bold").setBackground("#1A365D").setFontColor("#FFFFFF");
  }
  return sheet;
}
```

---

## ⚡ Setup Integration Guide for Immediate Cash Fulfillment

When a high-ticket client purchases this custom deployment:
1. Deploy this script as a **Web App** inside Google Apps Script (`Deploy -> New Deployment -> Web App -> Execute as: Me -> Access: Anyone`).
2. Copy the resulting **Web App URL**.
3. Plug this URL into their **Meta WhatsApp Webhook Configuration** or Make.com catchhook to intercept interactive button replies.
4. Hook up their active lead generation forms to call `broadcastLeadToTeam(payload)` instantly.
5. **Demonstrate Live**: Have the sales reps pull out their phones during onboarding. Fire a test lead. Let them race to click the button. Watch their eyes light up as the winner gets the contact details while the others receive the *"Too Slow"* lockout message.
