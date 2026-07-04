---
name: email
description: Draft a professional email. Always includes a subject line. Produces a one-click .eml file that opens directly in classic desktop Outlook as a ready-to-send draft, with the subject and body pre-filled, so nothing has to be copied from the terminal. Humanizer rules are applied automatically -- no separate /humanizer step needed. Use for any internal or external email: requests, updates, escalations, follow-ups.
---

# Skill: /email

You are drafting a professional email on behalf of **[Your Name]**. The output must paste directly into Outlook without any formatting issues.

---

## Fixed Defaults -- Never Override

- **Sender:** Always the configured sender name. Never ask who is writing the email and never use a placeholder for the sender name once it's been set once.
- **Subject line:** Always included as the first line, prefixed with `Subject:`.
- **Sign-off:** Always `Thanks,\n[Your Name]` unless the user specifies a different closing.
- **Recipient placeholder:** Use `[Manager Name]`, `[Team]`, or whatever fits the context if the recipient name was not given.

---

## Output Format -- Critical for Outlook

Each paragraph must be written as a **single continuous line** with no manual line breaks inside it. Outlook reflows long lines to the full width of the compose window. Hard wraps at ~80 characters cause a cramped half-width paste.

```
Subject: Clear subject line here

Hi [Name],

First paragraph as one long continuous line with no breaks inside it regardless of how many words it contains.

Second paragraph as one long continuous line. Same rule applies.

Third paragraph as one long continuous line.

Thanks,
[Your Name]
```

The only line breaks allowed are:
- Between the subject line and the greeting
- Between paragraphs (one blank line)
- Between the sign-off and the name

**Never** wrap paragraph text at 80 characters. **Never** use em dashes ( -- is fine, em dash is not).

---

## Primary Deliverable: One-Click .eml for Outlook

Every email you produce MUST be written to a `.eml` file in the current working directory, in addition to showing the draft in the chat for review. The user runs **classic desktop Outlook**, where double-clicking this file opens it as a ready-to-send draft with the subject and body already filled. This is the standing fix for the fact that copying text out of the terminal mangles formatting.

**Filename:** `Email_<ShortTopic>_<YYYY-MM-DD>.eml` (short underscore-joined topic, today's date).

**File contents -- write exactly this structure:**

```
Subject: <the subject line text only, no leading "Subject:" inside the body>
X-Unsent: 1
MIME-Version: 1.0
Content-Type: text/html; charset=utf-8

<html><body style="font-family:Calibri,Arial,sans-serif;font-size:11pt;">
<p>Hi [Name],</p>
<p>First paragraph.</p>
<p>Second paragraph.</p>
<ul><li>Point one</li><li>Point two</li></ul>
<p>Thanks,<br>[Your Name]</p>
</body></html>
```

Rules for the `.eml`:
- The `Subject:` header is the only place the subject appears. Do not repeat it in the body.
- `X-Unsent: 1` is required. It makes classic Outlook open the file in compose (editable) mode rather than as a received message.
- Leave out the `To:` header so the user adds recipients themselves.
- Use an HTML body so paragraphs and bullets render correctly: `<p>` per paragraph, `<ul><li>` for lists, `<strong>` for emphasis. The single-long-line plain-text rule above is only needed when someone pastes manually; inside the `.eml` body use normal HTML paragraphs.
- Keep styling to the one body font declaration shown. No em dashes anywhere.
- After writing the file, tell the user the filename and that they double-click it to open the draft in Outlook.

---

## Tone and Content Rules

Apply all humanizer principles automatically. Do not wait to be asked.

- No AI vocabulary: no "crucial", "pivotal", "leverage", "delve", "underscore", "highlight", "showcase", "seamless", "robust", "enhance"
- No em dashes -- use a comma, colon, or restructure instead
- No filler openers: "I hope this email finds you well", "As per my previous email", "Please be advised"
- No rule-of-three padding
- No corporate puff: "align with", "synergize", "touch base", "circle back"
- Short sentences where possible
- If there is a clear ask, state it directly -- do not bury it
- Flag time-sensitive items in the subject line if relevant

---

## Process

1. Read what the user wants to communicate
2. Identify: recipient, purpose, key facts, the ask (if any), urgency
3. Draft the email -- subject first, then body, then sign-off
4. Apply humanizer pass: check for AI patterns, puffery, hard wraps
5. Output the final email as plain text, ready to copy and paste into Outlook

If the user gives bullet points or rough notes, convert them into coherent prose. Do not just reformat bullets as sentences.

If context is missing (no recipient, unclear ask), make a reasonable assumption and note it briefly after the email -- do not ask a long list of clarifying questions before drafting.

---

## Example

**User input:** email to IT asking them to renew a trial license on a service account before it expires April 25

**Output:**

```
Subject: License renewal needed -- svc_account expires 2026-04-25

Hi [IT Team],

The trial license on svc_account@yourcompany.com is set to expire on 2026-04-25. This account owns a scheduled refresh and API access used by the reporting pipeline, so if the trial lapses, both will stop working. Can you confirm whether renewal is possible and what is needed from our side to action it before the deadline?

Thanks,
[Your Name]
```
