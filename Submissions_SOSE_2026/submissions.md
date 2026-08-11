# Proposal Submission — SOSE 2026

---

## Team Details 👥

**Mentor's Name:** Jim Anderson

**Team Members:**

| GitHub Username | Full Name |
|---|---|
| @Jextic | Johnny Huynh |

---

## p5.js Issue 📋

<!-- Details about the first issue assigned to your team. -->

- **Link to Issue**: [github.com/processing/p5.js/issues/8922](https://github.com/processing/p5.js/issues/8922)

- **Issue Title**: [p5.js 2.0+ Bug Report]: Loading indication #8922

- **Repository**: p5.js

---

## Abstract 📝

<!-- 3–5 sentences. What is the idea, why does it matter, and what do you plan to do? -->

p5.js has canvas description functions like describe() and describeElement() to generate invisible DOM containers with text that can be read by screen readers. These DOM elements don't have a lang attribute so the TTS engines of the screen readers will be read using the primary voice regardless of the text's language. If the screen reader's voice is defaulted to an English voice and the text is written in a non-English language, the TTS would mangle the pronunciations and produce unintelligible outputs. This could be solved by modifying the describe functions to accept an additional parameter which injects a defined lang attribute into the DOM containers.

---

## Problem Statement 📑

<!-- What problem are you solving and who is affected by it? -->

Users who rely on screen readers would find this very helpful. After testing sentences in various languages for Windows Narrator and NVDA screen reader, I discovered that sentences without the lang attribute will always use the primary screen reader voice. Oftentimes, the words are hard to understand if the screen reader voice's language is different from the text's language. Additionally, if there is non-Latin text, the screen reader will skip over that block of text entirely if not supported. For example, an English narrator will skip over a sentence written in Kanji or Devanagari. Solving this issue would improve and expand access for international users.

I've also noticed that this is primarily a Windows issue. MacOS VoiceOver has basic Unicode detection so it has the capability of guessing the text's language and switching to the correct narrator although I'm unsure how accurate the macOS VoiceOver usually is. Something to take note of is that if the screen reader doesn't support or doesn't have the language voice installed, it will still use the default voice instead. For the most part, users would already have the voice language that they want already installed and popular screen readers like NVDA already have support for tons of languages.

HTML Test used for Windows Narrator and NVDA:
```
<div aria-label="English No Lang">
  <p>this is a test</p>
</div>
<div aria-label="English With Lang" lang="en">
  <p>this is a test</p>
</div>
<div aria-label="Spanish No Lang">
  <p>esto es una prueba</p>
</div>
<div aria-label="Spanish With Lang" lang="es">
  <p>esto es una prueba</p>
</div>
<div aria-label="Vietnamese No Lang">
  <p>Cái này là bài thi</p>
</div>
<div aria-label="Vietnamese With Lang" lang="vi">
  <p>Cái này là bài thi</p>
</div>`
```

I tested this with the English voice as the primary voice. I made sure that the Spanish language/voice pack and Vietnamese language/voice pack were installed for both NVDA and Windows Narrator before testing. The div containers with the lang attribute were successfully switched over to the proper narrator voice while the ones without the lang attribute only used the English voice.

HTML Test in Hindi (Latin and Devanagari):
```
</div>
<div aria-label="Hindi Latin No Lang">
  <p>yah test hai</p>
</div>
<div aria-label="Hindi Latin With Lang" lang="hi">
  <p>yah test hai</p>
</div></div>
<div aria-label="Hindi Devanagari No Lang">
  <p>यह टेस्ट है</p>
</div>
<div aria-label="Hindi Devanagari With Lang" lang="hi">
  <p>यह टेस्ट है</p>
</div>`
```

This test was generated with the help of Google Translate so the sentence itself may not be correct. The two tests with the lang attribute were successfully switched over to the Hindi voice narrator. The 1st test was used with the English voice narrator. It sounded ok but if I replaced it with a random longer sentence, it sounds a bit more incomprehensible. After the 2nd test, it immediately skipped over to the 4th test. It was most likely because the English voice doesn't recognize Devanagari and cannot read it so it just skipped and continued at the next detectable sentence.

---

## Proposed Solution 💡

<!-- High-level description of your approach and why you chose it. -->

I propose to update the `describe()` and `describeElement()` functions in `p5.js/src/accessibility/describe.js` to accept an additional parameter which will define the lang attribute. Screen readers can already determine the values of the attribute so abbreviations like 'vi', 'es,', 'hi', and 'fr' can be detected as Vietnamese, Spanish, Hindi, and French. The `_describeHTML()` and `_describeElementHTML()` will also be updated accordingly so that if a lang parameter exists from the previous functions, it will inject a `lang='__'` into the div container. 

The parameter will also be optional so that existing p5.js sketches won't be affected. Adding the lang parameter also shouldn't affect existing parameters like `LABEL` or `FALLBACK` The reference documentations would also be updated to encourage users to write in the native script of their languages instead of the transliteration of that language in Latin characters.

---

## Research on old issues 🔭

<!-- If there are existing open issues that align with or validate your proposed idea, reference them here. Your proposal may also help revive or advance work that has stalled. If no directly related issue exists, demonstrate that you have researched the project's backlog by identifying similar or relevant issues. Include links to those issues and briefly explain. -->

- [Issue #4721: Web accessibility next steps [conversation]](https://github.com/processing/p5.js/issues/4721)

This was a conversation of the implementation of describe functions. The author of the issue brought up the question "what should we do about web accessibility in languages other than English". This issue is one of the ways that p5.js could solve to expand language accessibility, especially for visually impaired users.

- [Issue #6992: Accessibility Features Proposal - Expand Web Accessibility module](https://github.com/processing/p5.js/issues/6992)

Issue #6992 focuses more on improving existing accessibility features and covering the WCAG (Web Content Accessibility Guidelines) rules. Adding support for multilingual screen reader voice switching would adhere to [WCAG 3.1.2 Language of Parts](https://www.w3.org/TR/WCAG22/#language-of-parts)

---

## Impact 🛠️

<!-- Select all that apply and add a one-line explanation for each. -->

- [x] **Feature Implementation** — Adds an optional lang parameter for describe functions for multilingual screen reader support
- [ ] **Bug Fix** — 
- [ ] **Performance** — 
- [ ] **Scalability** —
- [X] **Documentation** — Update reference documentation to include a note about using native script instead of transliteration.
- [X] **Testing** — Add unit test cases to verify that the lang attribute is properly injected into div containers.
- [ ] **Other** —

---

## Inclusivity and Accessibility 🤝

<!-- Required for proposals that are not bug fixes. If this is a bug fix, write "Not applicable — bug fix".

*How does your proposal increase inclusivity or accessibility?* -->

This proposal would satisfy WCAG Criterion 3.1.2 Language of Parts which states that text can be programmatically determined assistive technologies like screen readers. This would help Windows users who use screen readers because without the lang attribute, TTS voices would not be able to detect or switch to the proper language voices.

---

## Implementation Plan ⏳

<!-- Week-by-week breakdown of how you plan to complete the work. -->

Week 1: Refactor describe functions in `p5.js/src/accessibility/describe.js` to accept an additional argument for lang attributes. Also update the HTML generator functions to inject the lang attribute. Test for functionality and create unit cases. Hopefully create a PR at the end of the week.

Week 2: Take into account feedback and add any suggested changes. Finalize the PR.

---

## Deliverables 📦

<!-- What will concretely exist at the end of the internship that does not exist today? -->

After adding the changes, the describe functions will have multilingual support for screen reader voice switching. `describe.js` functions would have support for an additional optional parameter for a lang attribute and the JSDOC reference comments would be updated.

<!-- Anything you'd like maintainers to know that doesn't fit above. -->


