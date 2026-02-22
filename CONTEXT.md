# CONTEXT — taxi-palmier-voice

## What this repo is
Voice AI dispatcher using Twilio ConversationRelay + ElevenLabs TTS + Claude.
This is the premium front door — every incoming call goes through here.

## What this service does
- Receives incoming calls via Twilio webhook
- Runs real-time conversation using ConversationRelay
- Speaks using ElevenLabs dynamic TTS (not pre-recorded MP3)
- Follows a strict dialog state machine
- Calls Claude API only for extraction — not for business logic
- Posts confirmed ride JSON to the backend dispatch engine

## Conversation State Machine
```
GREETING → COLLECTING → CLARIFYING → CONFIRMING → HANDOFF → ESCALATED
```

## Two Separate Claude Calls (never mixed)

### 1. Dialog Agent (inline, during call)
Persona: Marie, professional Taxi Palmier dispatcher
Language: Quebec French, warm, natural
Rules: one question at a time, max 2 sentences per response
Never: gives prices, promises exact ETA, improvises outside flow

### 2. Extraction Agent (after confirmation phrase)
Input: full conversation transcript
Output: structured JSON only
```json
{
  "pickup_text": "",
  "dropoff_text": "",
  "time_type": "now|scheduled",
  "scheduled_time": null,
  "customer_phone": "",
  "notes": "",
  "confidence": 0.0,
  "needs_followup": false,
  "followup_question": null
}
```
If confidence < 0.85 → trigger clarification loop
If confidence >= 0.85 → POST to backend /rides

## Full Stack
- Voice: Twilio + ConversationRelay + ElevenLabs + Claude (Sonnet 4.6) ← THIS REPO
- Backend: Node.js + TypeScript + Fastify on DigitalOcean
- Database: Supabase (Postgres + Realtime + Auth)
- Driver App: Expo React Native + Expo Push Notifications
- Dashboard: Next.js on Vercel

## Environment Variables
```
TWILIO_ACCOUNT_SID=
TWILIO_AUTH_TOKEN=
ELEVENLABS_API_KEY=
ELEVENLABS_VOICE_ID=
ANTHROPIC_API_KEY=
BACKEND_URL=
PORT=3001
```

## Current Build Phase
[ ] Twilio webhook receives call
[ ] ConversationRelay connected
[ ] ElevenLabs TTS working
[ ] Dialog agent prompt tested in Claude Console
[ ] Extraction agent prompt tested in Claude Console
[ ] State machine implemented
[ ] Confidence check + clarification loop
[ ] POST to backend on confirmed ride
[ ] Escalation path working
[ ] Tested with 20 real call scenarios

## Decisions Locked
- ElevenLabs for TTS — not pre-recorded audio
- Claude Sonnet 4.6 for both dialog and extraction
- Dialog and extraction are SEPARATE Claude calls
- State machine controls flow — LLM never makes business decisions
- Extraction confidence threshold: 0.85
- Escalation if system cannot resolve after 2 clarification attempts
