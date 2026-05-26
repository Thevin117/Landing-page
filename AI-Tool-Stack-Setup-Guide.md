# 🛠️ THE SYNAPRIX AI TOOL STACK
## Step-by-Step Setup Guide with Real Configurations

**This is the exact tech stack we use with clients. Everything here works, costs under RM500/month total, and can be deployed in 3-5 days.**

---

## 📋 THE COMPLETE STACK

| Tool | Purpose | Cost | Setup Time |
|------|---------|------|------------|
| **Vapi.ai** | Voice AI receptionist | $99-299/month | 2-3 hours |
| **Make.com** | Automation glue | $9-29/month | 1-2 hours |
| **Google Calendar** | Booking system | Free | 30 mins |
| **Twilio** | SMS confirmations | ~$20/month | 30 mins |
| **Airtable/HubSpot** | CRM | Free-$50/month | 1 hour |
| **Slack** | Team notifications | Free | 15 mins |

**Total:** ~RM300-500/month for a complete AI front desk

---

## 🎤 TOOL 1: VAPI.AI (AI VOICE RECEPTIONIST)

**What it does:** Answers phone calls 24/7, sounds human, books appointments

**Setup Process:**

### Step 1: Create Account
1. Go to vapi.ai
2. Sign up (they give $10 free credit to test)
3. Go to Dashboard → Assistants → Create New Assistant

### Step 2: Configure Your Assistant

**Basic Settings:**
```yaml
Name: [Your Business] Receptionist
Voice: "Jenny" (US English, friendly) or "Nova" (younger, energetic)
Model: GPT-4 (more natural) or GPT-3.5-turbo (cheaper)
Temperature: 0.7 (balanced between creative and consistent)
```

**System Prompt:**
[Copy the AI Receptionist prompt from the Prompt Templates document]

**First Message (greeting):**
```
Hi, thanks for calling [BUSINESS NAME]! This is [AI NAME]. How can I help you today?
```

### Step 3: Function Calling (for booking)

Vapi lets you call external APIs mid-conversation. Set this up for checking calendar availability:

**Function: check_availability**
```json
{
  "name": "check_availability",
  "description": "Check if a time slot is available for booking",
  "parameters": {
    "type": "object",
    "properties": {
      "date": {
        "type": "string",
        "description": "Date in YYYY-MM-DD format"
      },
      "time": {
        "type": "string", 
        "description": "Time in HH:MM format (24-hour)"
      },
      "service": {
        "type": "string",
        "description": "Type of service being booked"
      }
    },
    "required": ["date", "time"]
  }
}
```

**Function: create_booking**
```json
{
  "name": "create_booking",
  "description": "Create a new appointment booking",
  "parameters": {
    "type": "object",
    "properties": {
      "customer_name": {"type": "string"},
      "phone": {"type": "string"},
      "email": {"type": "string"},
      "date": {"type": "string"},
      "time": {"type": "string"},
      "service": {"type": "string"},
      "notes": {"type": "string"}
    },
    "required": ["customer_name", "phone", "date", "time", "service"]
  }
}
```

### Step 4: Get Your Phone Number

Vapi provides a phone number, or bring your own:
- **Vapi number:** $1/month, instant setup
- **Your existing number:** Forward calls to Vapi number (carrier dependent)

**Call Forwarding Setup:**
- **After hours only:** Forward from 6 PM - 9 AM + weekends
- **Overflow:** Forward when lines are busy (4+ rings)
- **Full replacement:** Forward all calls (test thoroughly first!)

### Step 5: Testing Checklist

Call your Vapi number and test these scenarios:
- [ ] General question (business hours, location)
- [ ] Booking request (should check calendar, confirm slot)
- [ ] Urgent issue (should offer to transfer or take callback number)
- [ ] Price question (should answer or route to human)
- [ ] Random small talk (should politely redirect to purpose of call)

**Common Issues & Fixes:**

**Issue:** AI talks too long
**Fix:** Add to prompt: "Keep responses under 2 sentences. Speak in 10-15 word chunks."

**Issue:** AI sounds robotic
**Fix:** Change voice to "Jenny" or "Nova", increase temperature to 0.8

**Issue:** AI doesn't understand accents
**Fix:** Add example phrases to prompt: "Customer might say 'appointment' as 'appoint-ment' or 'booking' - these mean the same thing"

---

## 🔗 TOOL 2: MAKE.COM (AUTOMATION HUB)

**What it does:** Connects Vapi → Calendar → CRM → Slack → SMS

**Setup Process:**

### Step 1: Create Account
1. Go to make.com
2. Sign up (free plan: 1,000 operations/month - enough to start)
3. Create a new Scenario

### Step 2: Scenario 1 - New Booking Flow

**Trigger:** Webhook (Vapi calls this when booking confirmed)

**Modules in sequence:**

1. **Webhook** (receives data from Vapi)
   ```json
   {
     "customer_name": "John Doe",
     "phone": "+60123456789",
     "email": "john@example.com",
     "date": "2026-06-15",
     "time": "14:00",
     "service": "Consultation"
   }
   ```

2. **Google Calendar: Create Event**
   - Summary: `[Service] - [Customer Name]`
   - Start: `[Date] [Time]`
   - Duration: 60 minutes (or your default)
   - Description: Include phone, email, notes

3. **Twilio: Send SMS** (to customer)
   ```
   Hi [Name]! You're booked at [Business] for [Service] on [Date] at [Time]. 
   Reply CANCEL to cancel. See you soon!
   ```

4. **Slack: Send Message** (to team channel)
   ```
   🎉 New booking!
   
   Customer: [Name]
   Service: [Service]
   When: [Date] at [Time]
   Phone: [Phone]
   
   Added to calendar ✓
   ```

5. **Airtable/HubSpot: Create Contact** (add to CRM)
   - Name: [Customer Name]
   - Phone: [Phone]
   - Email: [Email]
   - Tag: "AI_BOOKED"
   - Source: "AI_Receptionist"

**To connect Vapi to Make:**
1. In Vapi dashboard → Settings → Webhooks
2. Add Make.com webhook URL
3. Select trigger: "booking_completed"
4. Test by making a test booking call

### Step 3: Scenario 2 - Lead Scoring Flow

**Trigger:** Webhook (from WhatsApp chat, website form, etc.)

**Modules:**

1. **Webhook** (receives lead data)
2. **OpenAI: Create Completion** (score the lead)
   ```
   Analyze this lead and score them as HOT/WARM/COLD based on:
   - Problem urgency
   - Budget fit
   - Timeline
   
   Lead info: [paste webhook data]
   
   Output JSON: {"score": "HOT", "reason": "..."}
   ```

3. **Router** (split based on score)
   - Route 1: HOT → Slack alert + immediate callback
   - Route 2: WARM → Email sequence + CRM tag
   - Route 3: COLD → Newsletter + long nurture

4. **CRM Update** (tag lead with score)

### Common Make.com Mistakes to Avoid:

❌ **Don't:** Chain 20+ modules in one scenario (slows down, hard to debug)  
✅ **Do:** Break into multiple scenarios, trigger via webhooks

❌ **Don't:** Run on every single event (wastes operations)  
✅ **Do:** Batch process or add filters ("only if status = confirmed")

❌ **Don't:** Hardcode values  
✅ **Do:** Use variables and error handlers

---

## 📅 TOOL 3: GOOGLE CALENDAR (BOOKING SYSTEM)

**What it does:** Stores all appointments, syncs with team calendars

**Setup:**

1. Create a dedicated booking calendar (don't use personal)
2. Set working hours (Calendar Settings → Working Hours)
3. Add buffer times between appointments:
   - Settings → Event Settings → Default Event Duration: 60 mins
   - Add 15-min buffer by ending events at :45 instead of :00

**Integration with Vapi:**
1. Get Calendar API credentials (Google Cloud Console)
2. In Make.com, connect Google Calendar module
3. Vapi → Make → Calendar (creates event)

**Pro tip:** Create different calendars for different services
- calendar-consultations@yourbusiness.com
- calendar-treatments@yourbusiness.com
- Route bookings to the right calendar based on service type

---

## 💬 TOOL 4: TWILIO (SMS CONFIRMATIONS)

**What it does:** Sends booking confirmations, reminders, cancellation handling

**Setup:**

1. Sign up at twilio.com
2. Get a phone number ($1/month)
3. Copy Account SID and Auth Token
4. In Make.com:
   - Add Twilio module
   - Connect with SID + Token
   - Send SMS on booking confirmation

**SMS Templates:**

**Booking Confirmation:**
```
Hi [Name]! You're booked at [Business] for [Service] on [Date] at [Time].

Reply CANCEL to cancel.

See you soon!
- Team [Business]
```

**24-Hour Reminder:**
```
Reminder: You have an appointment tomorrow at [Time] for [Service].

Reply CONFIRM or CANCEL.

[Business Address]
```

**Cancellation Received:**
```
Your [Date] appointment has been cancelled. 

To rebook, call [Phone] or visit [Website].
```

**Pro setup:** Handle SMS replies in Make.com
- Webhook from Twilio (incoming SMS)
- If message contains "CANCEL" → delete calendar event, notify team
- If message contains "CONFIRM" → add confirmed tag to booking

---

## 📊 TOOL 5: AIRTABLE (CRM)

**What it does:** Stores all customer data, tracks lead pipeline

**Setup:**

**Base Structure:**

**Table: Customers**
| Field | Type | Purpose |
|-------|------|---------|
| Name | Single line text | Full name |
| Phone | Phone number | Contact |
| Email | Email | Contact |
| Source | Single select | AI_Receptionist, Website, Referral |
| Lead Score | Single select | HOT, WARM, COLD |
| Status | Single select | New, Contacted, Booked, Closed, Lost |
| First Contact | Date | When they first reached out |
| Notes | Long text | All conversation history |
| Lifetime Value | Currency | Total spent |

**Table: Bookings**
| Field | Type | Purpose |
|-------|------|---------|
| Customer | Link to Customers | Who booked |
| Service | Single select | What they booked |
| Date | Date | Appointment date |
| Time | Single line text | Appointment time |
| Status | Single select | Confirmed, Completed, Cancelled, No-show |
| Booking Source | Single select | AI, Manual, Website |

**Automations in Airtable:**
- When Status = "Booked" → Send to "Upcoming" view
- When Date is today → Move to "Today's Appointments" view
- When Status = "Completed" → Calculate LTV

**Connect to Make.com:**
1. Make.com → Airtable module
2. On booking → Create record in Bookings table
3. On new lead → Create record in Customers table

---

## 🔔 TOOL 6: SLACK (TEAM NOTIFICATIONS)

**What it does:** Instant alerts when hot leads come in or bookings happen

**Setup:**

1. Create channels:
   - `#ai-bookings` - all confirmed appointments
   - `#hot-leads` - urgent leads needing immediate callback
   - `#ai-logs` - full conversation logs (optional)

2. Create Slack webhook:
   - Slack → Apps → Incoming Webhooks
   - Add to channel
   - Copy webhook URL

3. In Make.com:
   - Add HTTP module
   - POST to webhook URL
   - Send formatted message

**Message Format for Hot Leads:**
```json
{
  "text": "🔥 HOT LEAD - CALL NOW",
  "blocks": [
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*New Hot Lead* 🔥\n\n*Name:* John Doe\n*Phone:* +60123456789\n*Problem:* Urgent issue with X\n*Budget:* In range\n*Next step:* Call within 1 hour"
      }
    },
    {
      "type": "actions",
      "elements": [
        {
          "type": "button",
          "text": {"type": "plain_text", "text": "Call Now"},
          "url": "tel:+60123456789",
          "style": "danger"
        }
      ]
    }
  ]
}
```

---

## 🚀 DEPLOYMENT ROADMAP

### Week 1: Core Setup
- [ ] Day 1: Set up Vapi, test voice calling
- [ ] Day 2: Configure prompts, add FAQs
- [ ] Day 3: Connect Google Calendar
- [ ] Day 4: Set up Make.com booking flow
- [ ] Day 5: Add Twilio SMS confirmations
- [ ] Day 6-7: Test everything with team, fix issues

### Week 2: Go Live
- [ ] Day 1: Forward after-hours calls only (6 PM - 9 AM)
- [ ] Day 2-3: Monitor all calls, tune prompts
- [ ] Day 4: Add CRM integration (Airtable)
- [ ] Day 5: Set up Slack notifications
- [ ] Day 6-7: Analyze first week data, optimize

### Week 3: Scale
- [ ] Add lead qualifier (WhatsApp or web chat)
- [ ] Connect lead scoring to CRM
- [ ] Build email nurture sequences
- [ ] Set up analytics dashboard

---

## 🛠️ TROUBLESHOOTING GUIDE

**"AI isn't booking appointments correctly"**
→ Check Make.com logs. Usually calendar permissions or wrong date format.

**"Customers complaining AI sounds robotic"**
→ Switch to "Jenny" or "Nova" voice, add more conversational fillers to prompt.

**"Too many bad leads getting through"**
→ Tighten qualifier questions, add explicit budget check earlier.

**"Team not getting notifications"**
→ Check Slack webhook URL, test with manual trigger in Make.com.

**"Calendar double-bookings"**
→ Add buffer times, check timezone settings, verify availability check works.

---

## 💰 COST BREAKDOWN (MONTHLY)

| Item | Cost |
|------|------|
| Vapi.ai (1000 mins/month) | $99-149 USD (~RM400-650) |
| Make.com (Basic plan) | $9 USD (~RM40) |
| Twilio (500 SMS) | $20 USD (~RM85) |
| Google Calendar | Free |
| Airtable (Plus plan) | $20 USD (~RM85) |
| Slack | Free |
| **TOTAL** | **~RM600-860/month** |

Compare to hiring a full-time receptionist: RM3,000-5,000/month

**ROI:** Pays for itself if you capture just 2-3 extra bookings per month.

---

## 📞 NEED HELP SETTING THIS UP?

We do this for clients all the time. If you want us to set up your entire AI stack:

**DM "SETUP" on Instagram @synaprix**

We'll jump on a free 30-min call and walk you through it step-by-step.

Or book it as a done-for-you service (we handle everything, you just approve the prompts).

---

Created by Synaprix AI  
Synaprix.com · @synaprix on Instagram

**License:** Free to use for your own business. Do not resell or repackage.
