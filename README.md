# Doctor Appointment README and LinkedIn Post

## Amazing title — use this everywhere

**I Built a Doctor's Receptionist That Never Sleeps — Book on WhatsApp, Pay Online, Get Confirmed in Seconds**

Short version for the GitHub repo title line and LinkedIn headline:

**Doctor Appointment System — WhatsApp Booking + Website Booking + Online Payment + Auto Confirmation**

How to use this file: Part 1 is the complete GitHub README — copy everything inside that section into the `README.md` in your repo. Part 2 is the complete LinkedIn post — copy it as-is under your edited demo video. Repo setup details are given first, so you do not have to set anything up from memory.

## GitHub repo setup

- **Repo name:** `doctor-appointment`
- **Repo link:** https://github.com/adilahmedmeraj53904/doctor-appointment
- **Description:** Doctor appointment booking system with WhatsApp AI booking, website booking, Stripe online payment, auto payment confirmation, cancel and reschedule — built with n8n, Next.js, Supabase and Stripe.
- **Live demo:** https://doctor-appointment-ten-ebon.vercel.app
- **Topics:** `n8n`, `nextjs`, `supabase`, `stripe`, `whatsapp-api`, `ai-agent`, `appointment-booking`, `workflow-automation`, `chatbot`, `typescript`
- **Visibility:** Public
- **Demo video:** Attach your edited 66-second screen recording (slot list → booking → Stripe test payment → "Payment Successful!") to the LinkedIn post. Keep the large video on LinkedIn / YouTube and link it from the README, instead of uploading a large file into the repo.

---

# Part 1 — GitHub README (copy-paste ready)

## Doctor Appointment System — WhatsApp Booking + Website Booking + Online Payment + Auto Confirmation

A full-stack appointment booking system for a General Physician in Rajlokkhi, Uttara, Dhaka. Patients can book in two ways — by chatting with an AI agent on WhatsApp, or from the website — pay online, and get an instant confirmation message. No phone call to the chamber, no paper serial list, and no double booking.

**Live demo:** https://doctor-appointment-ten-ebon.vercel.app

**Stack:** n8n · Next.js · TypeScript · Tailwind CSS · Supabase · Stripe · WhatsApp Business API · OpenRouter

## What the system does

### WhatsApp booking

- The patient says hi, hello, or salam, and the AI Agent starts the booking conversation.
- The agent asks one question at a time: date, slot, name, age, phone number, and health problem.
- It shows only the booking rules that apply: Saturday to Wednesday, 4:00 PM to 8:00 PM, Thursday and Friday closed.
- There are 12 slots a day, 20 minutes each — Serial 1 at 4:00 PM to Serial 12 at 7:40 PM.
- Patients can book up to 7 days ahead.
- Before saving, the system checks the slot. If the slot is already taken, booking is blocked.
- One phone number can hold only one active booking. A lookup runs before every save, so the same person cannot create duplicate active bookings.
- On confirmation, the booking is saved in Supabase and the patient gets a full confirmation message with serial, date, time, and payment status.

### Website booking

- Patients can book from the website instead of WhatsApp, choosing date and slot and entering the same details.
- My Appointments page: patients search by phone number to see their bookings.
- Each booking card shows serial, date, time, and payment status — Pending or Paid — with a Pay Now button.
- Patients can cancel and reschedule from My Appointments.
- A password-protected admin panel shows the doctor all appointments in one place, so the doctor does not need a separate calendar or register book.

### WhatsApp payment method

- After a WhatsApp booking is saved, n8n creates a Stripe Checkout Session from an HTTP Request node and sends the payment link inside the WhatsApp booking confirmation: "Pay now: " + the checkout URL.
- The patient pays $5.00 in Stripe test mode.
- The appointment ID is passed as Stripe metadata (`metadata[appointmentId]`), so the payment can be matched back to the exact booking.
- The website booking path also has Pay Now — on the booking confirmation and on My Appointments cards.

### Payment confirmation message

- A separate n8n workflow, "Stripe Payment Webhook", receives the Stripe webhook on the production URL: https://n8n.meraj.shop/webhook/stripe-payment
- It continues only when the event is `checkout.session.completed` and `payment_status` is `paid`.
- It looks up the appointment in Supabase by the metadata appointment ID, updates `payment_status` to `paid`, and sends the patient a WhatsApp message:

> Payment Successful!
>
> Patient: [patient name]
> Amount: $5.00
>
> Your appointment payment is complete. Thank you!

- The message is sent to the patient's actual WhatsApp chat number, saved in a dedicated `whatsapp_number` column at booking time — not just to the phone number typed in the booking form. The two can be different, and the confirmation has to reach the chat.

### Cancel and reschedule on WhatsApp

- Patients can cancel or reschedule in the same WhatsApp chat, without calling the chamber.
- The AI Agent uses its conversation memory and a JSON `action` field to route booking, cancel, and reschedule correctly.
- The system finds the patient's latest booked appointment by phone number, then cancels or moves it.
- Cancel and reschedule are allowed up to 2 hours before the appointment.
- This flow passed a full end-to-end test on 2026-11-14.

## Doctor and booking rules

| Item | Detail |
| --- | --- |
| Doctor | General Physician |
| Location | Rajlokkhi, Uttara, Dhaka |
| Chamber days | Saturday to Wednesday |
| Closed | Thursday and Friday |
| Chamber time | 4:00 PM to 8:00 PM |
| Slots per day | 12 slots, 20 minutes each |
| Serial range | Serial 1, 4:00 PM — Serial 12, 7:40 PM |
| Booking window | Up to 7 days ahead |
| Cancel / reschedule | Up to 2 hours before |
| Consultation fee shown | 500 tk |
| Online payment in demo | $5.00, Stripe test mode |

## How it works

### Website flow

Next.js website on Vercel → Supabase tables (`doctors`, `slots`, `appointments`, `patients`) → booking, My Appointments, admin panel, and Pay Now.

### WhatsApp flow

WhatsApp Trigger in n8n → AI Agent with Simple Memory → Supabase lookup and save → WhatsApp confirmation with Stripe payment link.

- Simple Memory uses the sender's phone number as the session key, so every patient gets their own conversation history.
- Context Window is set to 20, so the agent remembers the full booking, cancel, and reschedule conversation.
- The system message uses a dynamic date, so the agent always knows today's day and date and never offers a closed day or a past date.
- The agent collects details one at a time and outputs booking JSON only after the patient confirms with a final yes.

### Payment flow

Booking saved in Supabase → n8n HTTP Request creates a Stripe Checkout Session with the appointment ID in metadata → payment link sent on WhatsApp → patient pays → Stripe webhook to n8n → Supabase `payment_status` updated to `paid` → WhatsApp "Payment Successful!" message.

## Problems I faced — and how I solved them

### 1. WhatsApp messages did not reach n8n

**Problem:** Sending worked, but a patient's incoming "hi" never triggered the workflow. The Meta app was in Development mode and everything looked correct in n8n.

**Solution:** I re-checked the webhook and trigger setup step by step. From 2026-11-13 the WhatsApp Trigger started receiving messages correctly — green check with 1 item — and the full chat flow worked from there. Lesson: verify the trigger with a real incoming message before debugging the rest of the workflow.

### 2. The AI Agent forgot the conversation and mixed up patients

**Problem:** A bare "yes" meant nothing without context, and one shared memory would mix different patients together.

**Solution:** Simple Memory with the sender's phone number as the session key — one memory per patient — and Context Window raised from 5 to 20. Now "yes" is correctly treated as the answer to the agent's last question.

### 3. A bare number did not trigger a booking step

**Problem:** Typing a bare "4" did not move the flow forward, while "Slot 4" did.

**Solution:** The system message and agent instructions were tightened so slot selection is asked and confirmed explicitly, instead of relying on the patient guessing the exact format.

### 4. Stripe webhook returned 404

**Problem:** Stripe showed 404 errors for `checkout.session.completed`, `payment_intent.created`, and `payment_intent.succeeded` — even after a successful payment.

**Solution:** The Stripe endpoint was pointing at the n8n webhook-test URL. A test URL accepts only one event and then returns 404. I moved the endpoint to the production webhook URL, and deliveries returned 200 OK. Rule I follow now: always use the production URL for Stripe resends and live tests.

### 5. I was checking the wrong Stripe event

**Problem:** Charge events contain `paid: true` but no `payment_status`, so a check built on charge events could never match correctly.

**Solution:** The webhook workflow checks the data under `$json.body`, and continues only on `checkout.session.completed` where `payment_status` equals `paid`.

### 6. Supabase rejected the appointment ID — "invalid input syntax for type uuid"

**Problem:** Looking up the appointment by the ID from Stripe metadata failed with a UUID error. The value looked correct on screen, and there was no visible space to remove.

**Solution:** The ID coming from Stripe metadata could not be trusted as a clean UUID value, so I trimmed it in the Supabase filter expression — `{{ $json.body.data.object.metadata.appointmentId.trim() }}` — and the lookup worked. This was also an access lesson: the Supabase credential in n8n must be the Secret / Service Role key, not the publishable key.

### 7. The payment success message would have gone to the wrong number

**Problem:** The booking form phone number and the WhatsApp chat number can be different. Sending "Payment Successful!" to the form number could send the confirmation to a number that never chatted with the system.

**Solution:** I added a `whatsapp_number` column to the `appointments` table, saved the chat number directly from the WhatsApp Trigger (`messages[0].from`) when the booking was created, and sent the payment confirmation to that number.

### 8. Wrong success URL thinking

**Problem:** It was tempting to point Stripe back to the n8n webhook as the success URL.

**Solution:** Kept the two URLs separate for what they actually do: `success_url` is the website payment success page the patient sees, and the n8n webhook URL is only for Stripe's server-to-server event.

### 9. Admin access needed protection without overbuilding

**Problem:** The admin panel shows patient names and phone numbers, so it could not stay open — but building full Supabase Auth for a learning project was more than the project needed.

**Solution:** A simple password gate (Option A): an admin password environment variable, a login route, and a session check on the admin page. Login was tested live, and the password was later changed by me and redeployed on Vercel so the new password is live.

### 10. WhatsApp recipient restriction

**Problem:** Even after switching the Meta app to Live mode and adding a permanent System User token, WhatsApp still showed "Recipient phone number not in allowed list". The sender is still the Meta test number.

**Solution — honest status:** A permanent token fixes token expiry, and Live mode fixes app status — neither fixes the recipient list. The real fix is registering a real WhatsApp Business number. Until then, the system is fully tested with the Meta test setup.

## What I deliberately did not add

- **Appointment reminders** — deferred by choice. They are not essential for the first working version.
- **Google Calendar sync** — not needed. The admin panel already shows all appointments in one place.
- **Booked slots greyed out on the website** — booking a taken slot is already blocked at confirm time, which is where double booking actually happens.

## Current status

- Website booking, WhatsApp booking, cancel, and reschedule: working, tested end to end.
- Stripe test payment from WhatsApp, with auto `payment_status` update and WhatsApp confirmation: working perfectly — end-to-end test passed on 2026-11-15.
- Website Pay Now: built and test passed at $5.00.
- Stripe runs in test mode only. Stripe does not support Bangladeshi merchant accounts for live money, so SSLCommerz is the recommended option for a real clinic in Bangladesh.
- This is an educational and portfolio project. The Supabase setup used for learning has Row Level Security disabled — a production deployment needs RLS enabled.

## What I learned

- A booking system is not just a form. The hard parts are state: one slot, one patient, one active booking per phone, and a payment matched to exactly one appointment.
- Webhooks fail silently in useful ways — test URL versus production URL, and the wrong event type, cost more time than the payment code itself.
- Always save the identifier you will actually need later. The chat number, not just the form number, is what makes a confirmation message reach the right person.

---

# Part 2 — LinkedIn post (copy-paste ready)

Post this with your edited 66-second video. The video should show: slot list → WhatsApp booking → Stripe $5 test payment → "Payment Successful!" message. Do not show the admin panel, any password, token, or API key in the video.

```text
I built a doctor's receptionist that never sleeps. 🏥

No phone calls. No paper serial list. No double booking.

A patient can now book a doctor's appointment in two ways — and pay online without calling the chamber:

✅ WhatsApp booking
The patient chats on WhatsApp. The AI Agent asks date, slot, name, age, phone and problem — one question at a time — checks if the slot is free, and confirms the serial.

✅ Website booking
Same booking from the website, with a My Appointments page where patients find their booking by phone number and see Pending or Paid status.

✅ Pay on WhatsApp
Right after booking, the patient gets a Stripe payment link inside WhatsApp. One tap, pay, done.

✅ Instant confirmation message
Stripe tells my n8n workflow the payment is complete → the booking auto-updates to Paid in Supabase → the patient instantly gets "✅ Payment Successful!" on WhatsApp.

✅ Cancel & reschedule on WhatsApp
No call needed. The system finds the patient's booking by phone number and cancels or moves it.

✅ One phone = one active booking, and a taken slot can never be booked twice.

The problems that taught me the most:

→ My Stripe webhook kept returning 404. Reason: I was using the n8n test URL, which accepts only one event. Production URL fixed it.
→ Supabase rejected my appointment ID as an invalid UUID. Trimming the ID from Stripe metadata fixed the lookup.
→ My payment confirmation would have gone to the wrong number — the form phone and the WhatsApp chat number can be different. I added a separate whatsapp_number saved directly from the chat.
→ Charge events in Stripe have no payment_status. I had to listen for checkout.session.completed instead.

Stack: n8n + Next.js + Supabase + Stripe + WhatsApp Business API

Doctor hours built into the system: Sat–Wed, 4:00–8:00 PM, 12 slots a day, Thu–Fri closed.

This started as a learning project. It is now a working system I can offer to clinics and doctors.

Live demo and full README — links in the comments. 👇

#n8n #AIAutomation #NextJS #Supabase #Stripe #WhatsApp #BuildInPublic #Bangladesh
```

## First comment to add right after posting

Post the links as the first comment, not in the main post, so the post itself stays clean:

```text
Live demo: https://doctor-appointment-ten-ebon.vercel.app
GitHub: https://github.com/adilahmedmeraj53904/doctor-appointment
```

## Before you post — 30-second check

- Video shows no admin panel, patient phone numbers, password, token, or API key.
- You describe the payment honestly as Stripe test mode ($5.00 test payment).
- The GitHub README in the repo is this Part 1 version, with no real email, token, or secret key anywhere in the text or workflow files.
