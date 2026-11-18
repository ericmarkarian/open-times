# Open Times

**Check a meeting request against your Outlook calendar, without leaving the email.**

Open Times is a lightweight, custom Outlook add-in that reads the email you're viewing, interprets proposed dates and times, checks your calendar availability, and prepares a reply. It is designed for the small but repetitive task of answering emails like *“Are you free Thursday at 3?”*

## Features

- **In-Outlook workflow:** Open a received email and launch the **Am I free?** add-in task pane.
- **Meeting request parsing:** Extracts common dates, time ranges, and relative dates from the message, with manual edits if the interpretation needs correcting.
- **Availability verdict:** Shows **FREE**, **CONFLICT**, or availability windows for less-specific requests.
- **Daily timeline:** Displays busy periods and the requested meeting time.
- **Alternatives:** Suggests other available time slots when the proposed time conflicts.
- **Reply drafts:** Creates friendly, rule-based responses and opens a prefilled Outlook reply. No generative AI or paid model API is used.
- **Calendar hold shortcut:** Can open a prefilled Outlook appointment form; it does not silently create an event.

## How it works

```text
Received email in Outlook
        |
        v
Outlook add-in (Office.js + taskpane.html)
  - reads the open email
  - parses requested date/time
        |
        v
Cloudflare Worker (/api/calendar)
  - fetches the published Outlook ICS feed
  - parses calendar events and time zones
  - returns calendar time blocks
        |
        v
Open Times (client-side JavaScript)
  - checks conflicts and free slots
  - renders the day's timeline
  - prepares a reply in Outlook
```

**Technology:** Office.js, a sideloaded Outlook XML manifest, vanilla HTML/CSS/JavaScript, GitHub Pages for the task pane, and a Cloudflare Worker for retrieving the published calendar feed.

The Cloudflare Worker avoids cross-origin restrictions that prevent the GitHub-hosted page from fetching the Outlook ICS feed directly. The Outlook ICS URL is intended to be stored as a **Cloudflare secret** (`ICS_URL`), not in this public repository.

## Repository files

| File | Purpose |
| --- | --- |
| `taskpane.html` | Outlook task-pane interface, email parsing, availability calculation, and reply templates |
| `icon-32.png`, `icon-80.png` | Outlook add-in icons |
| `calendar-connection-test.html` | Diagnostic page used to test browser access to the ICS endpoint |

The **Cloudflare Worker** and **Outlook XML manifest** are separate deployment components and are not currently checked into this repository. The manifest must point to the hosted task pane.

## Running your own instance

This repository is currently configured as a **personal prototype**, not a multi-user service. Installing the same manifest without replacing its calendar backend would check the original owner's calendar, **not automatically the installer's calendar**.

To make a personal instance:

1. Publish an Outlook calendar as an ICS feed, using the minimum calendar-sharing detail you're comfortable with.
2. Deploy your own Cloudflare Worker configured to fetch and parse the feed. Set the published ICS URL as the Worker secret named `ICS_URL`.
3. Update the `WORKER_BASE` value in `taskpane.html` to the URL of your own Worker.
4. Host the task pane and icon files over HTTPS (for example, with GitHub Pages).
5. Create or update an Outlook XML add-in manifest pointing to the hosted task pane, then sideload it into Outlook using **Add a custom add-in → Add from File** (if your organization allows custom add-ins).
6. Open a received email and launch **Am I free?**.

There is no Microsoft Graph application registration in the current ICS-based architecture. The add-in does not require a generative AI service.

## Privacy and security

> [!WARNING]
> **The current no-sign-in Worker endpoint is public.** Anyone who discovers the endpoint can potentially query the calendar information it returns. Keeping `ICS_URL` as a Cloudflare secret protects the upstream URL from the frontend source code, **not the response data**.

- Prefer publishing **free/busy only** and returning anonymous busy blocks.
- **Do not expose event subjects, locations, descriptions, or other sensitive calendar details** through an unauthenticated Worker endpoint.
- Never commit published ICS URLs, API keys, access tokens, or credentials to GitHub.
- A published ICS URL functions like a shareable access link; rotate/unpublish it if exposed.
- Published ICS feeds can be delayed or incomplete. Verify availability against Outlook before relying on it for time-sensitive meetings.
- The current project is **not** a secure, authenticated multi-user calendar service.

## Limitations

- Date and time interpretation uses local rules rather than AI, so ambiguous requests may need correction.
- Calendar availability depends on the published ICS feed, which may not reflect changes immediately.
- Outlook add-in availability and sideloading depend on the Microsoft 365 organization's policies and client support.
- Reply drafts require user review; the add-in opens a reply rather than sending it automatically.
- Recurring meetings and time-zone definitions need to be handled correctly in the Worker to avoid false-free results.

## Status

**Personal proof of concept.** The add-in has been tested with Outlook and an ICS-backed Cloudflare Worker, but it is not packaged as a self-service product or independently security-audited.

---

Built by [Eric Markarian](https://github.com/ericmarkarian). Not affiliated with or endorsed by Microsoft or Harvard Medical School.
