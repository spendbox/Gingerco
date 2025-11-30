# Session 3 Email Configuration Guide

This document explains how to configure the Google Apps Script to send automated confirmation emails for Session 3 (Afrobeats Dance Workshop) registrations.

## Overview

When a user registers for Session 3, the system needs to:
1. Save their registration data to Google Sheets
2. Send an automated confirmation email with event details

## Google Apps Script Setup

### Current Setup
The registration form currently submits to:
```
https://script.google.com/macros/s/AKfycbxBCnq3023OZ-LCYXhQFRYFSfLmgXPKt3THAq1o-CrWc0t-NHvgHmMnOWC6ajnPmVo/exec
```

### Required Modifications

The Google Apps Script needs to be updated to handle email sending when `sendEmail: true` and `emailType: 'session3'` are present in the request data.

### Email Template

When Session 3 is selected, the script should send the following email:

**Subject:** Your Registration for the Afrobeats Dance Workshop – Confirmed!

**Body:**
```
Dear {fullName},

Thank you for registering for Session 3: Afrobeats Dance Workshop with Kafeela Ade at the Afrobeats Indoor Cycling: Vienna Takeover 2025.

We're excited to have you join us!

📅 Date: Saturday, 6 December
⏰ Session Time: 17:30
🏃‍♀️ Arrival: Please arrive 20 minutes early to check in and get settled.

📍 Location:
Sport Arena Wien
Stephanie-Endres-Straße 3, 1020 Wien, Austria

What to bring:
• Comfortable clothing and indoor shoes
• Water bottle
• Lots of energy!

Age Requirement:
Suitable for ages 6 and above.

If you have any questions before the event, feel free to reach out to us at events@gingerandco.at.

We're looking forward to dancing with you!

Warm regards,
The Ginger & Co. Events Team
```

### Implementation Code (Google Apps Script)

Add this to your existing `doPost(e)` function in Google Apps Script:

```javascript
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);

    // Save to spreadsheet (existing logic)
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    // ... existing spreadsheet logic ...

    // Check if email should be sent
    if (data.sendEmail === true && data.emailType === 'session3') {
      sendSession3Email(data);
    }

    return ContentService.createTextOutput(JSON.stringify({
      status: 'success'
    })).setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    Logger.log(error);
    return ContentService.createTextOutput(JSON.stringify({
      status: 'error',
      message: error.toString()
    })).setMimeType(ContentService.MimeType.JSON);
  }
}

function sendSession3Email(data) {
  const recipientEmail = data.email;
  const fullName = data.fullName;
  const subject = 'Your Registration for the Afrobeats Dance Workshop – Confirmed!';

  // Create HTML email body
  const htmlBody = `
    <!DOCTYPE html>
    <html>
    <head>
      <style>
        body { font-family: Arial, sans-serif; line-height: 1.6; color: #333; }
        .header { background-color: #D4AF37; padding: 20px; text-align: center; }
        .header img { max-width: 200px; }
        .content { padding: 20px; }
        .highlight { background-color: #f9f9f9; padding: 15px; border-left: 4px solid #D4AF37; margin: 15px 0; }
        .footer { background-color: #f4f4f4; padding: 15px; text-align: center; font-size: 0.9em; color: #666; }
      </style>
    </head>
    <body>
      <div class="header">
        <img src="https://res.cloudinary.com/dslj7k6lx/image/upload/v1763663531/Ginger_logo_1_act33r.png" alt="Ginger & Co. Logo">
        <h2 style="color: white; margin-top: 10px;">Ginger & Co.</h2>
      </div>

      <div class="content">
        <p>Dear ${fullName},</p>

        <p>Thank you for registering for <strong>Session 3: Afrobeats Dance Workshop with Kafeela Ade</strong> at the Afrobeats Indoor Cycling: Vienna Takeover 2025.</p>

        <p>We're excited to have you join us!</p>

        <div class="highlight">
          <p><strong>📅 Date:</strong> Saturday, 6 December</p>
          <p><strong>⏰ Session Time:</strong> 17:30</p>
          <p><strong>🏃‍♀️ Arrival:</strong> Please arrive 20 minutes early to check in and get settled.</p>
        </div>

        <p><strong>📍 Location:</strong><br>
        Sport Arena Wien<br>
        Stephanie-Endres-Straße 3, 1020 Wien, Austria</p>

        <p><strong>What to bring:</strong></p>
        <ul>
          <li>Comfortable clothing and indoor shoes</li>
          <li>Water bottle</li>
          <li>Lots of energy!</li>
        </ul>

        <p><strong>Age Requirement:</strong><br>
        Suitable for ages 6 and above.</p>

        <p>If you have any questions before the event, feel free to reach out to us at <a href="mailto:events@gingerandco.at">events@gingerandco.at</a>.</p>

        <p>We're looking forward to dancing with you!</p>

        <p>Warm regards,<br>
        <strong>The Ginger & Co. Events Team</strong></p>
      </div>

      <div class="footer">
        <p>© 2025 Ginger & Co. All rights reserved.</p>
        <p>For more information: <a href="mailto:info@gingerandco.at">info@gingerandco.at</a></p>
      </div>
    </body>
    </html>
  `;

  // Send email
  MailApp.sendEmail({
    to: recipientEmail,
    subject: subject,
    htmlBody: htmlBody,
    name: 'Ginger & Co. Events Team'
  });

  Logger.log(`Session 3 confirmation email sent to ${recipientEmail}`);
}
```

## Testing

To test the email functionality:

1. Register for Session 3 through the form
2. Check the registered email inbox for the confirmation
3. Verify all details are correct (name, date, location, etc.)

## Troubleshooting

If emails are not being sent:

1. **Check Google Apps Script execution logs:**
   - Go to Apps Script > Executions
   - Look for errors in the `sendSession3Email` function

2. **Verify email quota:**
   - Free Gmail accounts: 100 emails/day
   - Google Workspace accounts: 1,500 emails/day

3. **Check spam folder:**
   - Initial emails might be marked as spam

4. **Verify form data:**
   - Check that the form is sending `sendEmail: true` and `emailType: 'session3'`

## Important Notes

- The email is sent automatically when Session 3 is selected
- No email is sent for Sessions 1, 2, or 4 (they redirect to Wien Ticket)
- The email includes the Ginger & Co. logo and branded styling
- All email parameters are configurable in the script

## Contact

For technical support with the email setup, contact the development team.
