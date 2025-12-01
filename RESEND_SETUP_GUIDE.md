# Resend Email Integration Guide

## Overview
This guide shows you how to integrate Resend with your Google Apps Script to send emails from `events@gingerandco.at` with excellent deliverability.

## Why Resend?
- ✅ Built for transactional emails (confirmations, notifications)
- ✅ Excellent deliverability (better than generic Gmail)
- ✅ Free tier: 3,000 emails/month
- ✅ Beautiful analytics dashboard
- ✅ Easy domain verification
- ✅ Won't end up in spam

## Step 1: Sign Up for Resend

1. Go to [resend.com](https://resend.com)
2. Create a free account
3. You'll get 3,000 emails/month free (more than enough for your events)

## Step 2: Verify Your Domain

1. In Resend dashboard, go to **Domains**
2. Click **Add Domain**
3. Enter `gingerandco.at`
4. Resend will provide you with DNS records to add

### DNS Records You'll Need to Add:

Resend will give you records like these (exact values will be different):

```
Type: TXT
Name: _resend
Value: resend-domain-verification=xxx123xxx
```

```
Type: TXT  (SPF)
Name: @
Value: v=spf1 include:resend.com ~all
```

```
Type: TXT  (DKIM)
Name: resend._domainkey
Value: v=DKIM1; k=rsa; p=MIGfMA0GCSq...
```

5. Add these to your domain registrar (wherever you manage gingerandco.at DNS)
6. Wait for verification (usually 5-15 minutes)

## Step 3: Get Your API Key

1. In Resend dashboard, go to **API Keys**
2. Create a new API key
3. **IMPORTANT:** Copy and save it securely (you'll only see it once)

Example: `re_123abc456def789ghi012jkl345mno678`

## Step 4: Update Google Apps Script

### Option A: Using Apps Script Properties (Recommended)

Store your API key securely:

1. In Apps Script, go to **Project Settings** (gear icon)
2. Scroll to **Script Properties**
3. Add property:
   - **Property:** `RESEND_API_KEY`
   - **Value:** Your Resend API key

### Complete Apps Script Code:

```javascript
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);

    // Save to spreadsheet (existing logic)
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
      Logger.log('Email sent successfully to ' + recipientEmail);
      Logger.log('Response: ' + responseBody);
    } else {
      Logger.log('ERROR: Failed to send email. Status: ' + responseCode);
      Logger.log('Response: ' + responseBody);
    }
  } catch (error) {
    Logger.log('ERROR sending email via Resend: ' + error.toString());
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

## Step 5: Test the Integration

1. Deploy your Apps Script as a web app
2. Submit a test registration for Session 3
3. Check your Resend dashboard to see the email delivery
4. Check your recipient inbox

## Monitoring & Analytics

Resend provides a beautiful dashboard where you can:
- See all sent emails
- Track delivery status
- View open rates (if enabled)
- See bounce/spam reports
- Download logs

## Troubleshooting

### Email not sending?

1. **Check Apps Script logs:**
   ```
   Apps Script Editor → Executions → View logs
   ```

2. **Verify API key:**
   - Make sure it's correctly set in Script Properties
   - Try generating a new one

3. **Check domain verification:**
   - Go to Resend dashboard → Domains
   - Ensure status is "Verified"

4. **Check Resend logs:**
   - Resend dashboard → Logs
   - See exact error messages

### Still going to spam?

1. **Warm up your domain:** Start with small volumes
2. **Check SPF/DKIM:** Ensure DNS records are correct
3. **Add DMARC record:**
   ```
   Type: TXT
   Name: _dmarc
   Value: v=DMARC1; p=none; rua=mailto:events@gingerandco.at
   ```

## Cost Breakdown

- **Free tier:** 3,000 emails/month
- **Pro tier:** $20/month for 50,000 emails
- **For your event:** If you expect 100-300 Session 3 registrations, free tier is perfect!

## Comparison: Resend vs Gmail

| Feature | Resend | Gmail (via Apps Script) |
|---------|--------|------------------------|
| Daily limit | 3,000 (free) | 1,500 (G Suite) |
| Deliverability | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Analytics | ✅ Full dashboard | ❌ None |
| Spam rate | Very low | Moderate |
| Setup time | 30 min | 15 min |
| Cost | Free | $6-12/month |

## Next Steps

1. Sign up for Resend
2. Verify your domain
3. Update Apps Script with code above
4. Test with a few registrations
5. Monitor in Resend dashboard

## Support

- Resend docs: https://resend.com/docs
- Contact: support@resend.com
