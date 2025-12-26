---
layout: post
title: "Zero Knowledge Email aka zkEmails"
date: 2025-12-26
---

Honestly, this whole thing started *accidentally*.

I was setting up **signup and notification emails** for a personal project. Nothing fancy, just the usual "thank you for signing up" stuff. I quickly *ChatGPT-ed* (*yes, I use this term now*) for a list of providers that support a free email-sending tier. I picked **Brevo** because they allow **300 emails per day** on the free plan.

While wiring this up, I ran into a surprisingly annoying problem.

I logged into the Brevo dashboard and created an SMTP credential. The UI asked me to copy the `apiKey` during creation, clearly stating it *won't be shown again*. Fair enough that's obviously the **password**.

But then… what's the **username**?

I confidently assumed `apiName` would work. Copied it, configured everything, and started testing.

As you might have already guessed —> **"Authentication Failed."**

Back to the dashboard. I stared at the UI again, looking for *anything* that resembled a username.

Time for trial and error.

```bash
openssl s_client -connect {brevo_smtp_host}:{port} -crlf -quiet
EHLO <custom-domain>.com # domain authenticated with Brevo
AUTH LOGIN
{username}
{password}
QUIT
```

Eventually, I discovered that a gnarly string like  
`<identifier>@smtp-brevo.com` labeled **Login** in the UI worked perfectly. Now that I'm writing this and looking back at the UI, I feel stupid.  
I don't know why it did not click immediately.

Anyway, reconfigured everything, and it worked like a charm. **Phew.**

Work done. Went out for dinner.

---

### The thought that wouldn't leave me

While sitting in the cab, I was reading through some zero-knowledge articles and papers I had collected during a math course on **elliptic curves**. The first few lectures were pure *"WTF is happening?"* but then gradually things started to make sense.

And that's when the obvious question hit me:

**Why don't we have end-to-end encryption for email?** I know some of you folks will point me to ProtonMail but bear with me :) 


More specifically:

- Why doesn't *my* Gmail have E2E encryption?
- Why should Gmail read **everything**?
- And why have we collectively accepted this for decades?

To make matters worse, Sergey is back, Gemini is being pushed hard, and let's be honest—that means *even more* of our emails will be read, indexed, and mined.

So I decided to build something for myself.

---

## Zero Knowledge Emails (zkEmails)

I came back home and started dusting off my old SMTP and IMAP knowledge. I wrote small snippets of code to:

- manually send emails using SMTP
- list messages using IMAP
- inspect headers
- see how providers actually store and expose messages

After a few hours of tinkering, ChatGpting and debugging, I had a working prototype.

## Introducing zkEmails (because why not?)

The idea is fairly simple:

- **Gmail remains the provider**
- SMTP is the mode of Transport for Sending emails
- IMAP is the mode of Transport for accessing Inbox
- No convincing friends to switch providers

Instead:

> A **thin client** sits *above* SMTP and IMAP and guarantees **end-to-end encryption**.

Our friend Gmail here just does the following:
- stores encrypted blobs
- delivers encrypted payloads

But **never** sees:

- message content
- attachments
- conversation history

I am going to post the code on my github soon with a detailed README and simplified setup strict.
