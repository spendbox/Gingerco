# Resend Setup with world4you Domain

## Your Setup

- 🌐 **Domain:** gingerandco.at (registered with world4you)
- 💻 **Website:** Hosted on GitHub
- 📧 **Email:** Will be sent via Resend API

**Important:** Your website can stay on GitHub! You only need to add DNS records at world4you to enable email sending.

---

## Part 1: Sign Up for Resend (5 minutes)

### Step 1: Create Resend Account

1. Go to [resend.com](https://resend.com)
2. Click "Sign Up"
3. Create your account (use events@gingerandco.at if you have it, or any email)
4. Verify your email

**Cost:** FREE for up to 3,000 emails/month (perfect for your event!)

---

## Part 2: Add Domain to Resend (5 minutes)

### Step 2: Add Domain in Resend

1. Log in to Resend dashboard
2. Click **"Domains"** in the left sidebar
3. Click **"Add Domain"**
4. Enter: `gingerandco.at`
5. Click **"Add"**

Resend will now show you DNS records that need to be added. Keep this page open!

---

## Part 3: Add DNS Records at world4you (10 minutes)

### Step 3: Log in to world4you

1. Go to [world4you.com](https://www.world4you.com)
2. Log in to your account
3. Go to **Domain Administration** / **Domain-Verwaltung**
4. Select `gingerandco.at`
5. Click on **DNS Settings** / **DNS-Einstellungen**

### Step 4: Add Resend DNS Records

Resend will give you **3 DNS records** to add. Here's how to add each one in world4you:

#### Record 1: Domain Verification (TXT)

**From Resend:**
```
Type: TXT
Name: _resend
Value: resend-domain-verification=abc123xyz (your actual value will be different)
```

**In world4you DNS panel:**
1. Click **"Add DNS Record"** / **"DNS Eintrag hinzufügen"**
2. Select record type: **TXT**
3. **Name/Host:** `_resend`
4. **Value/Data:** Copy the full verification string from Resend
5. **TTL:** Leave default (usually 3600)
6. Click **Save** / **Speichern**

#### Record 2: SPF Record (TXT)

**From Resend:**
```
Type: TXT
Name: @ (or leave blank)
Value: v=spf1 include:resend.com ~all
```

**In world4you DNS panel:**
1. Click **"Add DNS Record"**
2. Select record type: **TXT**
3. **Name/Host:** `@` (or leave blank if that's not allowed)
4. **Value/Data:** `v=spf1 include:resend.com ~all`
5. **TTL:** Leave default
6. Click **Save**

**Note:** If you already have an SPF record, you need to modify it instead:
- Find existing TXT record with `v=spf1`
- Add `include:resend.com` before the `~all` or `-all`
- Example: `v=spf1 include:resend.com include:_spf.google.com ~all`

#### Record 3: DKIM Record (TXT)

**From Resend:**
```
Type: TXT
Name: resend._domainkey
Value: v=DKIM1; k=rsa; p=MIGfMA0GCSqGS... (long string)
```

**In world4you DNS panel:**
1. Click **"Add DNS Record"**
2. Select record type: **TXT**
3. **Name/Host:** `resend._domainkey`
4. **Value/Data:** Copy the entire DKIM value from Resend (it's a very long string starting with `v=DKIM1`)
5. **TTL:** Leave default
6. Click **Save**

### Step 5: Verify Domain in Resend

1. Go back to Resend dashboard
2. Wait 5-15 minutes for DNS propagation
3. Click **"Verify"** next to your domain
4. If successful, you'll see a green checkmark ✅

**Troubleshooting:** If verification fails:
- Wait another 10 minutes (DNS can take time)
- Check that you entered the records exactly as shown
- Make sure there are no extra spaces in the values

---

## Part 4: Get API Key (2 minutes)

### Step 6: Create API Key

1. In Resend dashboard, go to **"API Keys"**
2. Click **"Create API Key"**
3. Give it a name: `Event Registration Emails`
4. Select permission: **"Sending access"**
5. Click **"Create"**
6. **IMPORTANT:** Copy the API key and save it somewhere safe!
   - It looks like: `re_123abc456def789ghi012jkl345mno678`
   - You'll only see it once!

---

## Part 5: Update Google Apps Script (10 minutes)

### Step 7: Add API Key to Apps Script

1. Open your Google Apps Script project
2. Click the **⚙️ gear icon** (Project Settings) on the left
3. Scroll to **"Script Properties"**
4. Click **"Add script property"**
5. **Property:** `RESEND_API_KEY`
6. **Value:** Paste your Resend API key
7. Click **"Save script properties"**

### Step 8: Update Your Script Code

Replace your existing `doPost` function with this complete code:

```javascript
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);

    // Save to spreadsheet (your existing logic)
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    sheet.appendRow([
      data.timestamp,
      data.fullName,
      data.email,
      data.phone,
      data.location,
      data.sessions,
      data.groupSize,
      data.ageConfirm,
      data.physicalReadiness,
      data.liabilityWaiver,
      data.eventRecording,
      data.stayConnected,
      data.gdprConsent,
      data.mediaConsent
    ]);

    // Check if email should be sent for Session 3
    if (data.sendEmail === true && data.emailType === 'session3') {
      sendSession3EmailViaResend(data);
    }

    return ContentService.createTextOutput(JSON.stringify({
      status: 'success'
    })).setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    Logger.log('Error: ' + error.toString());
    return ContentService.createTextOutput(JSON.stringify({
      status: 'error',
      message: error.toString()
    })).setMimeType(ContentService.MimeType.JSON);
  }
}

function sendSession3EmailViaResend(data) {
  const recipientEmail = data.email;
  const fullName = data.fullName;

  // Get API key from Script Properties
  const RESEND_API_KEY = PropertiesService.getScriptProperties().getProperty('RESEND_API_KEY');

  if (!RESEND_API_KEY) {
    Logger.log('ERROR: RESEND_API_KEY not found in Script Properties');
    return;
  }

  // Create HTML email body
  const htmlBody = createEmailHTML(fullName);

  // Prepare Resend API request
  const payload = {
    "from": "Ginger & Co. Events Team <events@gingerandco.at>",
    "to": [recipientEmail],
    "subject": "Your Registration for the Afrobeats Dance Workshop – Confirmed!",
    "html": htmlBody
  };

  const options = {
    "method": "POST",
    "headers": {
      "Authorization": "Bearer " + RESEND_API_KEY,
      "Content-Type": "application/json"
    },
    "payload": JSON.stringify(payload),
    "muteHttpExceptions": true
  };

  try {
    const response = UrlFetchApp.fetch("https://api.resend.com/emails", options);
    const responseCode = response.getResponseCode();
    const responseBody = response.getContentText();

    if (responseCode === 200) {
      Logger.log('✅ Email sent successfully to ' + recipientEmail);
      Logger.log('Response: ' + responseBody);
    } else {
      Logger.log('❌ ERROR: Failed to send email. Status: ' + responseCode);
      Logger.log('Response: ' + responseBody);
    }
  } catch (error) {
    Logger.log('❌ ERROR sending email via Resend: ' + error.toString());
  }
}

function createEmailHTML(fullName) {
  return `
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Arial, sans-serif;
      line-height: 1.6;
      color: #333;
      margin: 0;
      padding: 0;
      background-color: #f4f4f4;
    }
    .email-container {
      max-width: 600px;
      margin: 0 auto;
      background-color: #ffffff;
    }
    .header {
      background-color: #D4AF37;
      padding: 30px 20px;
      text-align: center;
    }
    .header img {
      max-width: 180px;
      height: auto;
    }
    .header h1 {
      color: #ffffff;
      margin: 15px 0 0 0;
      font-size: 24px;
      font-weight: 600;
    }
    .content {
      padding: 30px 20px;
    }
    .highlight-box {
      background-color: #f9f9f9;
      padding: 20px;
      border-left: 4px solid #D4AF37;
      margin: 20px 0;
    }
    .highlight-box p {
      margin: 8px 0;
    }
    .info-section {
      margin: 20px 0;
    }
    .info-section strong {
      color: #D4AF37;
    }
    ul {
      padding-left: 20px;
    }
    ul li {
      margin: 8px 0;
    }
    .footer {
      background-color: #f4f4f4;
      padding: 20px;
      text-align: center;
      font-size: 14px;
      color: #666;
    }
    .footer a {
      color: #D4AF37;
      text-decoration: none;
    }
    @media only screen and (max-width: 600px) {
      .content {
        padding: 20px 15px;
      }
      .highlight-box {
        padding: 15px;
      }
    }
  </style>
</head>
<body>
  <div class="email-container">
    <!-- Header with Logo -->
    <div class="header">
      <img src="https://res.cloudinary.com/dslj7k6lx/image/upload/v1763663531/Ginger_logo_1_act33r.png" alt="Ginger & Co. Logo">
      <h1>Ginger & Co.</h1>
    </div>

    <!-- Email Content -->
    <div class="content">
      <p>Dear <strong>${fullName}</strong>,</p>

      <p>Thank you for registering for <strong>Session 3: Afrobeats Dance Workshop with Kafeela Ade</strong> at the Afrobeats Indoor Cycling: Vienna Takeover 2025.</p>

      <p>We're excited to have you join us!</p>

      <!-- Event Details -->
      <div class="highlight-box">
        <p><strong>📅 Date:</strong> Saturday, 6 December 2025</p>
        <p><strong>⏰ Session Time:</strong> 17:30</p>
        <p><strong>🏃‍♀️ Arrival:</strong> Please arrive 20 minutes early to check in and get settled.</p>
      </div>

      <!-- Location -->
      <div class="info-section">
        <p><strong>📍 Location:</strong></p>
        <p>
          Sport Arena Wien<br>
          Stephanie-Endres-Straße 3<br>
          1020 Wien, Austria
        </p>
      </div>

      <!-- What to Bring -->
      <div class="info-section">
        <p><strong>What to bring:</strong></p>
        <ul>
          <li>Comfortable clothing and indoor shoes</li>
          <li>Water bottle</li>
          <li>Lots of energy!</li>
        </ul>
      </div>

      <!-- Age Requirement -->
      <div class="info-section">
        <p><strong>Age Requirement:</strong></p>
        <p>Suitable for ages 6 and above.</p>
      </div>

      <p>If you have any questions before the event, feel free to reach out to us at <a href="mailto:events@gingerandco.at" style="color: #D4AF37;">events@gingerandco.at</a>.</p>

      <p style="margin-top: 30px;">We're looking forward to dancing with you!</p>

      <p>
        Warm regards,<br>
        <strong>The Ginger & Co. Events Team</strong>
      </p>
    </div>

    <!-- Footer -->
    <div class="footer">
      <p>© 2025 Ginger & Co. All rights reserved.</p>
      <p>For more information: <a href="mailto:info@gingerandco.at">info@gingerandco.at</a></p>
    </div>
  </div>
</body>
</html>
  `;
}
```

### Step 9: Deploy Your Script

1. Click **"Deploy"** → **"New deployment"**
2. Click the gear icon ⚙️ next to "Select type"
3. Choose **"Web app"**
4. Settings:
   - **Description:** "Session 3 Email Integration"
   - **Execute as:** Me
   - **Who has access:** Anyone
5. Click **"Deploy"**
6. Copy the web app URL (it should be the same as your existing one)

---

## Part 6: Test the Setup (5 minutes)

### Step 10: Send a Test Email

1. Go to your event registration page
2. Fill out the form and select **Session 3**
3. Submit the registration
4. Check the Apps Script execution log:
   - Apps Script Editor → **Executions** (left sidebar)
   - Look for "✅ Email sent successfully"

### Step 11: Verify in Resend Dashboard

1. Go to Resend dashboard → **Emails**
2. You should see your test email listed
3. Check the status (should be "Delivered")

### Step 12: Check Your Inbox

Check the email address you used for registration. You should receive the beautiful confirmation email!

---

## 📊 What You Get with Resend

### Dashboard Analytics
- See all emails sent
- Track delivery status
- View bounce/spam reports
- Download logs

### Email Deliverability
- Industry-leading inbox placement
- Professional email infrastructure
- Automatic retry on failures
- Real-time delivery tracking

---

## 🔧 Troubleshooting

### Domain Not Verifying?

**Check world4you DNS:**
1. Go to world4you → DNS Settings
2. Make sure all 3 records are added correctly
3. Check for typos or extra spaces
4. Wait up to 30 minutes for propagation

**Verify DNS propagation:**
- Use [whatsmydns.net](https://www.whatsmydns.net)
- Enter `_resend.gingerandco.at`
- Select TXT record type
- Check if it shows globally

### Email Not Sending?

**Check Apps Script logs:**
1. Apps Script → Executions
2. Look for errors in red
3. Check if API key is set correctly

**Check Resend logs:**
1. Resend dashboard → Logs
2. Look for your email attempt
3. See specific error message

### Email Going to Spam?

**Initial emails might go to spam. To prevent this:**
1. Add `events@gingerandco.at` to contacts
2. Mark the first email as "Not Spam"
3. After a few successful sends, deliverability improves

**Add DMARC record (optional but recommended):**
```
Type: TXT
Name: _dmarc
Value: v=DMARC1; p=none; rua=mailto:events@gingerandco.at
```

---

## 💰 Cost Summary

- ✅ **Resend:** FREE (3,000 emails/month)
- ✅ **world4you:** No additional cost (just your domain registration)
- ✅ **GitHub hosting:** FREE
- ✅ **Google Apps Script:** FREE

**Total additional cost: €0** 🎉

---

## 🎯 Key Points

1. ✅ Your website stays on GitHub (no changes needed)
2. ✅ DNS records are managed at world4you
3. ✅ Emails are sent via Resend API from Apps Script
4. ✅ Everything is FREE for your event size
5. ✅ Professional deliverability from `events@gingerandco.at`

---

## 📞 Need Help?

- **world4you DNS issues:** [world4you support](https://www.world4you.com/support)
- **Resend setup:** [Resend docs](https://resend.com/docs) or support@resend.com
- **Apps Script errors:** Check execution logs in Apps Script editor

---

## ✅ Checklist

- [ ] Created Resend account
- [ ] Added domain to Resend
- [ ] Added 3 DNS records at world4you
- [ ] Domain verified in Resend (green checkmark)
- [ ] Created API key in Resend
- [ ] Added API key to Apps Script properties
- [ ] Updated Apps Script code
- [ ] Deployed web app
- [ ] Tested with registration
- [ ] Verified email delivery

Once all checked, you're ready to go live! 🚀
