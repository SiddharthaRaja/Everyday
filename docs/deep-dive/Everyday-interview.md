# Everyday ("Daily Forge") — Interview Questions & Answers

> Answers are first-person and take about 30–90 seconds to say. **[E]/[M]/[H]** = difficulty. Line numbers refer to `index.html` unless another file is named. *(verified)* = I ran it.

**Beginner primer.** This is one web page with 72 questions written into it. Each day it shows one question for each of five difficulty levels. You reveal the answer, and if you do all five in a day your streak goes up. A helper script is *meant* to let it work without internet.

---

## (a) Overview

**1. [E] What is Daily Forge?**
It's a daily "cognitive workout" app: five problems a day, one each in Easy, Medium, Hard, Impossible and Abyss, across 13 subjects from math to ethics. You reveal the answer and explanation, and finishing all five tiers keeps your streak alive.

**2. [E] Why build it?**
I wanted a habit that trains reasoning broadly, not just coding. Most brain-training apps are speed games. I wanted problems that make you think for five minutes: Fermi estimates, the decoy effect, the Prisoner's Dilemma, open problems.

**3. [E] Stack?**
One HTML file with vanilla JavaScript, `localStorage`, a web manifest and a service worker. Google Fonts for typography. No framework, no backend.

**4. [M] How much content is there?**
72 problems: Easy 17, Medium 18, Hard 14, Impossible 13, Abyss 10 *(verified by counting)*. Each has a category, prompt, hints, answer and explanation.

**5. [M] What's the idea behind the "Impossible" and "Abyss" tiers?**
They're genuinely unsolved or contested questions: P vs NP, the measurement problem, Newcomb's paradox, the Repugnant Conclusion. The reveal says "Framework" instead of "Answer" (`:440`). The point is practising how to reason when no answer exists.

---

## (b) Design

**6. [M] How do you pick today's problems?**
`idx` is the day of the year. For each tier I take `pool[idx % pool.length]` (`:374-378`), so everyone gets the same set on a given date and each tier cycles independently.

**7. [H] How often does the same set of five repeat?**
Each tier on its own repeats every n days (10 for Abyss). Because the pool sizes are 17, 18, 14, 13 and 10, the exact five-problem combination only repeats after their LCM, 278,460 days. In practice that doesn't matter, because day-of-year resets every January 1st. I'd switch to days since a fixed epoch.

**8. [M] How does the streak work?**
When a reveal makes all five tiers done for today, I check whether `lastFullDate` was yesterday. If so the streak goes up by one, otherwise it resets to 1. `longest` tracks the maximum (`:380-398`).

**9. [M] Why require all five tiers for the streak?**
To push people into the harder tiers instead of farming Easy. The trade-off is that 4 out of 5 counts as nothing, which could be discouraging. A partial-credit or per-tier streak would be gentler.

**10. [M] How does `render` work?**
It rebuilds the whole card as one template string assigned to `#root.innerHTML`, then re-attaches click listeners (`:411-491`). That's simple for a page this small. All problem text goes through `esc()`, which escapes HTML using `textContent` (`:405-409`).

---

## (c) Bugs & code-level

**11. [H] Does offline mode work?**
No, and I found out why. The manifest, the favicon links and the service worker's asset list all reference `icon-192.png` and `icon-512.png`, but the repo only has `ico192.png`. `cache.addAll` is atomic, so the 404 rejects the install, and the worker never activates.
- In headless Chromium: no registration, an empty cache, and an offline reload fails.
- With correctly named icons on a copy: it installs and caches everything *(both verified)*.

And because registration has `.catch(() => {})`, nobody would notice.

**12. [M] What's the streak display bug?**
The stored streak only changes when you complete a day. If you miss a week, the header still shows the old streak until your next full day, when it resets to 1. I reproduced it: a streak of 10 last completed in 2020 still showed 10. The display should be derived: 0 if `lastFullDate` is older than yesterday.

**13. [M] What happens if the app stays open past midnight?**
`today` and `idx` are computed once at load (`:368-369`). Revealing after midnight logs to the previous day and shows yesterday's problems. On phones, PWAs often stay alive in the background, so this is realistic. The fix is to recompute on every action or on `visibilitychange`.

**14. [H] Any time-zone or DST issue?**
Yesterday is `Date.now() − 86,400,000` (`:388`). On a 23- or 25-hour DST day, just after midnight that can land on the wrong date and break a valid streak. Calendar arithmetic, `new Date(y, m, d − 1)`, avoids it. Also, `dayIndexUTC` is misnamed: it feeds *local* date parts into `Date.UTC`, so it's really the local day of the year. That's what I want, but the name misleads.

**15. [M] Do the fonts work offline?**
No. The service worker's cross-origin branch is network-first with a cache fallback, but nothing ever writes fonts *into* the cache, so offline you get Georgia/monospace fallbacks. I'd use runtime caching or self-host the fonts.

**16. [E] Any manifest inconsistencies?**
The description lists four tiers ("Easy, Medium, Hard, Impossible") but the app has five, and both icon paths are missing.

---

## (d) Content quality

**17. [M] How did you make sure the answers are right?**
I wrote and reviewed them by hand. For this write-up I numerically checked the computable ones: Collatz from 27 takes 111 steps and peaks at 9,232; the Rule of 72 at 10% gives 7.2 against an exact 7.27; the birthday paradox at 23 people is 50.7%; the momentum and inscribed-square answers are right *(verified)*. The open-ended ones state that there's no consensus rather than inventing a "correct" answer.

**18. [M] Any weaknesses in the content?**
- Some hints give the answer away. The Doppler question's hint literally says "This is the Doppler effect".
- A few factual framings, like Netflix's tier labelling, have no source.
- The pools are small, so Abyss repeats every 10 days.

I'd add a `source` field and grow the pools.

**19. [M] How would you scale content?**
Move it to JSON files per tier with an id, version and source. Possibly accept community submissions with review. Optionally use an LLM to draft new problems, but with mandatory human verification of the answers.

---

## (e) UX & product

**20. [M] How do you know someone actually thought about it rather than just tapping reveal?**
I don't. The streak measures reveals, not effort. I'd add a self-grade ("got it / partly / missed") or, for tiers with numeric answers, an input box checked against the answer. Then show accuracy per category.

**21. [M] What does a user see after missing days?**
The stale streak described above. Better: "Your 10-day streak ended. Start a new one today."

**22. [E] Accessibility?**
Contrast is generally decent on the dark palette. The tier tabs are `div`s with click handlers, not buttons, so they aren't keyboard-focusable. I'd change them to `<button>` with `aria-pressed`.

**23. [M] Why localStorage and no account?**
Zero friction and full privacy. The cost is that there's no cross-device sync and clearing browser data loses your history. An export/import would be a cheap middle ground.

---

## (f) Testing

**24. [E] Tests?**
None in the repo. I used Playwright afterwards to verify the service-worker failure, the offline reload and the stale streak.

**25. [M] What tests would you write?**
- A Playwright test asserting the SW activates and an offline reload works. That would have caught the icon bug immediately.
- Streak transition tests with a mocked clock: consecutive days, a gap, the DST boundary, midnight with the app open.
- A content lint: every problem has all five fields and a known category symbol.

**26. [M] How would you mock time?**
Inject a `now()` function instead of calling `new Date()` directly, or use Playwright's clock API (`page.clock`) to set the system time.

---

## (g) Performance & architecture

**27. [E] Any performance concerns?**
No. 72 problems is a few tens of KB. Re-rendering via `innerHTML` replays the card animation on every tab switch, which is cosmetic.

**28. [M] Would you add a framework?**
Not for this size. If it grew to stats pages, history and settings, I'd use something light like Preact or Lit.

**29. [M] Why cache-first for your own files?**
Instant load, and it works offline once installed. The catch is that updates need a `CACHE_NAME` bump, and users see the new version one load later. I'd add an "update available, reload" prompt.

---

## (h) Behavioural (STAR)

**30. [M] Tell me about a bug you found only by testing the unhappy path.**
- **S:** The app claims "Works offline" in its footer.
- **T:** Confirm that claim.
- **A:** I loaded it in headless Chromium, inspected service-worker registrations and caches, and toggled offline. Nothing was cached. The network log showed a 404 for `icon-192.png`. I then renamed the icons on a copy and re-ran: the SW activated and cached everything.
- **R:** Root cause proven, with a five-minute fix.
- **Learning:** never swallow registration errors, and test the offline path explicitly.

**31. [M] Tell me about a product decision you'd revisit.**
- **S:** I made the streak require all five tiers.
- **T:** Encourage hard thinking.
- **A:** Looking at it again, a stale display plus an all-or-nothing rule means a user who misses one tier sees no progress, and sees a wrong streak number afterwards.
- **R:** I'd add per-tier streaks or partial credit, plus an honest "streak ended" message.
- **Learning:** habit apps live or die on how they handle failure days.

**32. [M] How did you choose the subjects?**
Breadth: math, physics, logic, history, business, economics, biology, systems, philosophy, marketing, strategy, ethics and estimation. Thirteen categories so that a week covers very different kinds of reasoning.

---

## (i) Extensions

**33. [M] What would you add next?**
Fix offline, fix the streak display, add self-grading, and add a history view to re-read past problems.

**34. [M] How would you add sharing?**
A "share today's Abyss question" button using the Web Share API with the prompt text. Since every user gets the same set each day, it naturally supports "did you solve today's?" conversations.

**35. [H] How would you personalise difficulty?**
Track self-grades per category and tier, and pick from the pool with spaced repetition: show missed problems again after 3, 7 and 21 days, and prefer categories with lower accuracy. That changes selection from a deterministic modulo to a seeded, per-user choice.

**36. [M] Could you add notifications?**
Yes. A daily reminder through the Notifications API from the service worker, scheduled with Periodic Background Sync where supported. That needs a working service worker first.

**37. [M] How would you internationalise it?**
Move all strings and content into locale JSON files, and format dates with `Intl`. The content translation is the big cost.

**38. [M] Would an LLM grader help for the open tiers?**
Possibly as feedback: "here's what a strong answer would consider". But it should never decide the streak. Open questions don't have a right answer, and the grader could be confidently wrong.

**39. [E] Hosting?**
GitHub Pages or any static host. HTTPS is needed for the service worker.

**40. [M] How would you measure whether it helps people?**
Opt-in, privacy-preserving metrics: retention (day-7 and day-30 streaks), self-grade accuracy over time, and which tiers get skipped. Before adding any analytics I'd ask users.

---

## Hostile follow-ups

**H1. "Your footer says 'Works offline'. It doesn't."**
Correct. A misnamed icon makes the service-worker install fail, and I swallowed the error. I've proven both the cause and the fix. The lesson is to test the claim you print.

**H2. "This is just a list of trivia in an HTML file."**
The engineering is small, I agree. The work is in curating graded reasoning problems with honest framing for the open ones. I'd present it as a product and content exercise, not a systems project.

**H3. "Your streak can be gamed by tapping reveal five times."**
Yes. It measures engagement, not correctness. A self-grade or answer input is the next feature.

**H4. "You named a function `dayIndexUTC` but it's local time. Sloppy?"**
The behaviour is what I want, the user's local day, but the name is wrong and misleading. I'd rename it and add a test around midnight.

**H5. "Why should I trust the answers?"**
The computable ones I've numerically checked. The contested ones explicitly say there's no consensus. The gap is sources for factual claims, which I'd add.

**H6. "No tests, no README. Was this rushed?"**
It was a quick personal build in one day, which the four commits show. Since then I've tested it and found three real bugs, which is exactly the argument for tests.

**H7. "Isn't 10 Abyss problems too few for a daily app?"**
Yes. They repeat every 10 days. Growing the pools, or making repeats intentional through spaced repetition, would fix that.

**H8. "What happens to user data if they clear the browser?"**
It's lost. That's the trade-off of no accounts. An export/import button is a cheap fix.

---

## Glossary (quick)

- **Atomic:** all-or-nothing; one failure cancels the whole operation.
- **Runtime caching:** caching files the first time they're fetched, instead of at install.
- **Spaced repetition:** reviewing items at growing intervals to remember them.
- **`visibilitychange`:** a browser event fired when the app is shown or hidden.
- **Web Share API:** opens the phone's native share sheet from a web page.
