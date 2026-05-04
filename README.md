# Case Study: Asura Tracker – Manga Reading Companion Chrome Extension

**Personal project** · Chrome Extension · React · TypeScript · Tailwind CSS

---

A Chrome extension for tracking manga reading progress on [AsuraComic.net](http://AsuraComic.net). It automatically logs which chapter you're on, checks for new releases in the background, and can navigate you to the next chapter automatically when you finish one. Available on the Chrome Web Store.

The interesting parts weren't the UI — they were the constraints that come with building inside a browser extension: no persistent server, no direct DOM control, three isolated execution contexts that all need to stay in sync, and a host site you don't own that can change at any time.

---

## The Interesting Problems

**Three contexts, one consistent state**

Chrome extensions run code in three separate environments that can't directly talk to each other: the background service worker, content scripts injected into the page, and the popup UI. All three need to agree on what you've read and what's new.

The solution was treating Chrome's local storage as the single source of truth and building a message-passing layer on top of the Chrome extension API to sync state across contexts. The popup doesn't fetch data itself — it reads from storage. The background worker writes to storage when it finds updates. The content script writes when you read a chapter. Everything stays consistent without any of the contexts needing to know about each other directly.

<img width="323" height="600" alt="Popup UI showing tracked series list" src="https://github.com/user-attachments/assets/9eb39170-08c4-41ac-9f71-81295026484d" />

**Detecting when someone finishes a chapter**

To trigger the auto-next feature, the extension needs to know when you've actually finished reading — not just scrolled to the bottom or switched tabs. A simple scroll percentage check produces too many false positives.

The final approach layered three signals: an Intersection Observer watching for the "Related Series" section that appears at the end of every chapter, a scroll depth threshold (90%+ of the document), and a minimum time-on-page requirement. All three have to agree before the extension considers a chapter finished. This eliminated false triggers from people who scroll through quickly or land on the wrong page.

**Background update checking without hammering the server**

The extension checks for new chapters on followed series in the background, but doing that on a fixed interval for everyone wastes requests and battery. The scheduler adapts based on your reading behavior — series you've read recently get checked more often, series you haven't touched in weeks get checked less. Requests are also batched and cached with a short TTL so opening the popup doesn't trigger a fresh network call every time.

**Building on a site you don't control**

Content scripts inject into a live website that can update its HTML structure at any time. Any selector that worked yesterday might break tomorrow. The chapter detection logic uses multiple fallback strategies rather than depending on a single CSS class or element ID — if the primary detection method fails, it tries the next one. This made the extension meaningfully more resilient to site changes without requiring a manual update.

<img width="317" height="597" alt="Series detail view with chapter history and controls" src="https://github.com/user-attachments/assets/63376e43-36bc-4451-a6c2-e400c10e8862" />

---

## Outcome

Published on the Chrome Web Store. Tracks reading progress across series, sends notifications when new chapters drop, and handles the chapter-to-chapter navigation flow automatically. All data stays local — nothing leaves the browser.
