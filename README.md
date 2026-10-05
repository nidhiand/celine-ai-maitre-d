# Céline: A Bilingual AI Maître d'

Céline is a bilingual (English and French) voice agent built on [ElevenAgents](https://elevenlabs.io). She answers the phone for Bistro Marceline, the flagship restaurant of Rive Hospitality Group, a fictional New York restaurant group.

I built her to understand ElevenAgents the way a customer would: starting from a business problem, making deployment tradeoffs, and testing the agent as if it were going live.

## The business problem

- **Missed calls cost bookings.** During dinner service, staff are busy and phones go unanswered.
- **Callers speak different languages.** A New York restaurant gets calls from tourists and locals alike.
- **Hospitality is revenue.** A good wine recommendation raises the check. A guest who is turned away without an alternative is lost.

## What Céline does

- Takes reservations, collecting date, time, party size, name and callback number, then reads the booking back for confirmation
- Recommends dishes and wine pairings from the restaurant's knowledge base
- Switches between English and French to match the caller
- Discloses at the start of every call that she is a virtual host
- Is designed to point guests to a sister restaurant when the flagship is closed or full
- Transfers to a manager for large parties, private dining, severe allergies and complaints
- Flags problem calls for staff review

## Design decisions and tradeoffs

| Decision | Choice | Why |
|---|---|---|
| Knowledge base | Full document included in the prompt, not retrieval | The guide is small (about 6 kB), and getting allergens and hours right matters more than efficiency. A restaurant group with dozens of menus would need retrieval instead. |
| LLM | Fast, low-cost hosted model | On a phone call, latency is the experience. I chose the model with the tightest response-time range rather than the most powerful one. |
| Reasoning effort and temperature | Low reasoning, low temperature | The job is conversational and rule-bound. Extra reasoning adds pauses, and high temperature risks improvised prices or policies. |
| Escalation | Warm (conference) transfer to a manager | The caller is never dropped onto a ringing line. |
| Allergies | Always recorded, never guaranteed | The kitchen handles common allergens, so the agent never promises a dish is allergen-free and offers a manager for severe cases. |
| Upselling | One natural wine suggestion, dropped if declined | Revenue without pushiness. |
| AI disclosure | Stated in the first message | Some jurisdictions require it, and honesty builds trust. |
| Silence handling | Soft timeout with a filler phrase | Prevents dead air during slower responses. |
| Language | French added as a second language, with automatic language detection and a French first message | Callers can switch languages mid-call without asking. |

## Testing

I ran test calls in the ElevenAgents preview and reviewed each transcript for correctness, rule-following and latency.

**What worked in the first test**

- Correctly resolved "this Saturday" to a specific date using the system time variable
- Collected reservation details in order and noted a birthday
- Recognized a severe allergy, offered a manager and transferred after the caller agreed
- Filled a slow response with "One moment" instead of silence
- Typical response latency of about 1 to 2 seconds per turn

**What broke, and how I fixed it**

| Issue | Fix |
|---|---|
| Asked the occasion question twice | Added a rule: one question per turn, never re-ask something the caller already answered |
| Repeated the caller's full name, which sounded robotic | Use only the first name, and only occasionally |
| Transferred without saying whether the table was held | Before any transfer during a booking, summarize what was noted and confirm the table is held |

## Next steps

- Compare three LLMs on latency, rule-following and cost per call using the same test scripts
- Test the closed-day redirect to sister restaurants and the mid-call language switch
- Add success criteria and data collection (party size, occasion, allergies, language) in ElevenAgents analysis
- Rebuild as a multi-step workflow with separate nodes for reservations, menu and wine, and escalation
- Add a webhook tool so confirmed reservations are written to a live booking sheet

## Files

- `marceline_system_prompt.md`: the agent's system prompt
- `marceline_knowledge_base.md`: the restaurant group's house guide (hours, policies, menu, wine list and pairings)

All restaurant names, addresses and phone numbers are fictional.
