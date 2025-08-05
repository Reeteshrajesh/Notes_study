# 📞 Twilio IVR – Alert Acknowledge/Resolve System

A simple IVR system using **Twilio Voice** that allows a caller to:
- Press **4** to acknowledge an alert
- Press **6** to mark an alert as resolved

Useful for monitoring systems, on-call alerts, support scenarios, etc.

---

## ✅ Goal

> Let a user interact with a voice prompt using their keypad and trigger different backend actions based on what they press.

- `4` → Acknowledge alert
- `6` → Resolve alert

---

## 🧰 Technologies Used

- Twilio Programmable Voice
- TwiML (Twilio Markup Language)
- Node.js + Express (API server)
- Optional: Database (for updating alert status)

---

## 🛠️ TwiML (Voice Prompt XML)

This XML response asks the user to press a key and sends the result to `/handle-key`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Response>
  <Gather action="/handle-key" method="POST" numDigits="1">
    <Say>Press 4 to acknowledge. Press 6 to resolve.</Say>
  </Gather>
  <Say>We didn't receive any input. Goodbye!</Say>
</Response>
````

---

## ⚙️ Backend Handler – `/handle-key`

This Express route receives the key pressed and sends a different voice response based on the digit.

```js
const express = require('express');
const bodyParser = require('body-parser');
const app = express();

app.use(bodyParser.urlencoded({ extended: false }));

app.post('/handle-key', (req, res) => {
  const digit = req.body.Digits;
  let responseText = "";

  if (digit === '4') {
    responseText = "Alert acknowledged. Thank you.";
    // TODO: update database to mark as acknowledged
  } else if (digit === '6') {
    responseText = "Alert resolved. Thank you.";
    // TODO: update database to mark as resolved
  } else {
    responseText = "Invalid input.";
  }

  res.type('text/xml');
  res.send(`
    <Response>
      <Say>${responseText}</Say>
      <Hangup/>
    </Response>
  `);
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server listening on port ${PORT}`);
});
```

---

## 🌐 Twilio Setup

1. **Buy a Twilio phone number**
2. Go to [Console > Phone Numbers](https://www.twilio.com/console/phone-numbers/incoming)
3. Under **Voice & Fax**, set:

   * **Webhook URL** to: `https://your-domain.com/ivr`
   * Method: `POST`

> If you're using ngrok locally, point to `https://<ngrok-id>.ngrok.io/ivr`

---

## 🔁 Flow Summary

```text
Incoming Call
     ↓
TwiML IVR Prompt → "Press 4 to acknowledge, 6 to resolve"
     ↓
User presses key
     ↓
POST /handle-key → process logic → response
     ↓
Call ends with message
```

---

## ✅ Optional Improvements

* Add timeout and retry logic in `<Gather>`
* Store call SID, user phone, and pressed digit to DB
* Support multiple digit inputs (e.g., "1234#")

---

## 📚 References

* [Twilio Programmable Voice Docs](https://www.twilio.com/docs/voice)
* [TwiML Reference](https://www.twilio.com/docs/voice/twiml)

---

## 🧪 Test with Ngrok

```bash
ngrok http 3000
```

Then use the generated HTTPS URL in Twilio's webhook settings.

---

## 📝 To Do

* [ ] Connect to a real database
* [ ] Authenticate incoming requests if needed
* [ ] Handle errors and logging

