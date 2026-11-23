---
name: spoken-replies
description: Write replies that are spoken aloud and heard rather than read. Load when writing a reply that a speech voice will say or a person will listen to, such as a voice assistant answer.
---

# Spoken replies

A listener hears a reply once, in order, and cannot scroll back or skim ahead. Write the way a clear person talks: the answer first, plain words, one idea at a time.

## Rules

1. **Lead with the answer.** The first sentence answers the question. Context comes after, if it is needed at all.
   - Instead of: "I looked through the build history for the last week and compared the runs. The build is passing."
   - Say: "Yes, the build is passing."
2. **Summarize at a high level, then offer to go deeper.** Give the gist, not every detail. Name what you could say more about and let the listener choose.
   - Instead of: a full account of every changed file.
   - Say: "Mostly small fixes to the login flow. I can go through the fixes or the tests, if you like."
3. **Use short sentences, and never read a list aloud.** One idea per sentence. Turn a list into a count and a summary, then name only the one or two items that matter.
   - Instead of: "The failures are: login test, signup test, reset test, billing test."
   - Say: "Four tests failed, mostly around accounts."
4. **Say numbers, dates and times the way people say them.** Round when the exact figure does not matter. Use relative days and spoken clock times.
   - Instead of: "2,013 users", "2026-12-01" and "15:30".
   - Say: "about two thousand users", "next Tuesday" and "half past three".
5. **Leave out Markdown, IDs, paths and links.** Nothing should only make sense written down. No headings, bullets, bold, code or symbols. Describe what a thing is instead of spelling out its identifier.
   - Instead of: "See `src/auth/session.ts`, ticket ABC-1234, at https://example.com/abc".
   - Say: "It's in the session code, in the ticket about expired logins."
6. **Ask conversational questions, not system prompts.** Ask the way a person would. Do not tell the listener which words to say back.
   - Instead of: "Say yes to confirm."
   - Say: "Does that sound good?"

## Checking a reply

Before speaking, read the draft as if you heard it once.

- Does the first sentence answer the question?
- Would a listener follow it without seeing it?
- Is anything in it that only works written down?
