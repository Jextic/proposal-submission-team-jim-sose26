# Proposal Submission Template — SOSE 2026

> Copy this file, rename it `proposal-team-"mentor's name".md`, and place it in the `Submissions SOSE 2026` folder. Do not raise a PR until your proposal is ready for final review. All the items marked with an asterisk (*) are required.

---

## Team Details* 👥

**Mentor's Name:** Jim Anderson

**Team Members:**

| GitHub Username | Full Name |
|---|---|
| @Jextic | Johnny Huynh |

---

## p5.js Issue* 📋

*Details about the first issue assigned to your team.*

- **Link to Issue**: [github.com/processing/p5.js/issues/8922](https://github.com/processing/p5.js/issues/8922)

- **Issue Title**: [p5.js 2.0+ Bug Report]: Loading indication
 #8922


- **Repository**: p5.js

---

## Abstract* 📝

*3–5 sentences. What is the idea, why does it matter, and what do you plan to do?*

In p5.js v2.0+, a default canvas is created as a guard in case the user calls on drawing methods before defining a canvas. More often than not, users will define their own canvas using `createCanvas()` which destroys the default canvas and creates a new canvas. This process seems redundant, can waste memory if the user is switching from 2D to WebGL context, and can cause a visual flicker when the default canvas is being replaced with the new one. I propose deferring the default canvas creation until right after setup() is completed or until it is explicitly needed by using a canvas tracking variable, getter functions to guard against edge cases, and a check for if a canvas was created.
---

## Problem Statement* 📑

*What problem are you solving and who is affected by it?*

The resources that are used to generate the default canvas would be wasted if the canvas is usually deleted right after. It also doesn't occur with just users creating a new canvas. The default canvas has a 2D rendering context but there are many users who want to create a canvas with 3D/WebGL context. As far as I know, canvas contexts can't be changed from 2D to 3D. It is required to create a new canvas for 3D visual artworks which will inevitably destroy the default canvas.

The p5.js editor makes the default canvas basically invisible by setting setting the canvas background to the same color as the preview color (when the user changes the theme to dark mode or high contrast mode). However, p5.js sketches might also be embedded in other websites, such as blogs or portfolio websites. If the webpage loads slowly because of asset loading or if the html background of the webpage is not monochromic, a visual flicker is more likely to occur and will be more noticeable. Users with low-end devices or users in heavy network traffic areas are also likely to experience this visual flicker if they run content-heavy sketches. To improve inclusivity and accessibility, resolving the issue of unnecessary resource allocation will improve user experience, especially for low-end device users.

---

## Proposed Solution* 💡

*High-level description of your approach and why you chose it.*

I propose to defer the canvas creation until it is explicitly needed. In p5.js/src/core/main.js, a 100x100 canvas is created in `async #_setup()`. Users also usually create a canvas in their sketch code inside of `function setup() {...}`. The default canvas created during `async #_setup()` will be removed and a canvas tracking boolean (for example, `this._canvasWasCreated = false`) will declared in the constructor to track if the the user has defined a canvas in their sketch. Since the canvas creation is done inside p5.js/src/core/rendering.js, `fn.createCanvas` and `fn.noCanvas()` can be updated to set the boolean to true. After `await context.setup()` completes in `async #_setup()`, p5 will check the canvas tracking boolean and if it is false, that means the user didn't define their own canvas so the default canvas will be created. If it is true, that means the user already defined their own canvas so there is no need to create a default canvas. 

It's very possible that p5.js beginner's might call on drawing functions, like `background()` or `circle`, or read canvas properties in the `functio setup() {...}` in their sketch code. To handle these edge cases, a helper function (like `this._canvasExists()`) will be implemented to check the canvas tracking boolean. If the boolean is false, the default canvas will be created immediately to prevent any canvas errors or crashes. Getter functions will be added/modifed for functions like `canvas` and `drawingContext` so that if the tracking the helper function will be called on and handle the relevant function calls. This will guaratee that early draw function calls and any addons inspecting canvas properties will still be compatible.

Since the tracking boolean will is declared in the constructor, that means that the proposed solution should work well with instance mode. If multiple sketches are created, they should each have their own canvas tracking boolean. From what I've seen as well, the internally registered addons should not be affected since the addons primarily use the presetup and postsetup lifecycle hooks. I'm not taking into account loading.js since our team is still working on it and we are still looking for a solution to perfect how the loading indicator will appear.

---

## Research on old issues* 🔭

*If there are existing open issues that align with or validate your proposed idea, reference them here. Your proposal may also help revive or advance work that has stalled. If no directly related issue exists, demonstrate that you have researched the project's backlog by identifying similar or relevant issues. Include links to those issues and briefly explain.*

- [Issue #8922: \[p5.js 2.0+ Bug Report\]: Loading indication](https://github.com/processing/p5.js/issues/8922):
This was the previous issue that our team worked on (and are still improving on). Working on this helped me learn about lifecycle hooks and was what intrigued me to look more into the how the canvas is created. We were thinking about creating a temporary canvas on top of the default/user canvas so the indicator would appear at the center of that canvas. This issue proposal was thought of when we were determining how we would approach the placement of the loading indicator.

- [Issue #177: createCanvas: problem with the default canvas](https://github.com/processing/p5.js/issues/177):
After looking through the issues for quite a while, I found that this issue was discussed a long time ago. That means that this issue is one that the maintainers have already looked into and found a patch a long time ago. However, this solution is over a decade old and the p5.js has evolved a lot of the years. Simply resizing the default canvas instead of destroying/replacing wouldn't work since we have different canvases as well. We can't resize a canvas with 2D context into canvas with 3D WebGL context. Additionally, resizing the canvas does nothing if the user calls on `noCanvas()` since the default canvas will still be created and immediately destroyed. Modern p5.js is much different from p5.js back then so that's why I propose to defer the canvas creation until it is necessary instead of resizing the canvas.



## Impact* 🛠️

*Select all that apply and add a one-line explanation for each.*

- [ ] **Feature Implementation** —
- [X] **Bug Fix** — Ensures that the visual flicker won't occur and won't affect page layouts.
- [X] **Performance** — Saves memory since it'll avoid an unnecessary creation of the default canvvas.
- [ ] **Scalability** —
- [ ] **Documentation** —
- [X] **Testing** — Covers all edge cases (setup(), noCanvas(), createCanvas(), 3D/WebGL canvas, instance mode, non-visual scripts, etc) and implements helper functions to guard against early canvas-related calls in the sketch code.
- [ ] **Other** —

---

## Inclusivity and Accessibility 🤝

> Required for proposals that are not bug fixes. If this is a bug fix, write "Not applicable — bug fix".

*How does your proposal increase inclusivity or accessibility?*

Not applicable - bug fix.

---

## Implementation Plan* ⏳

*Week-by-week breakdown of how you plan to complete the work.*

Week 6: Share the proposal with the maintainers and take into account any feedback or suggested changes. If it is accepted, I'll work on it as soon as possible. If this proposal is not something the maintainer's want to focus on, I'll continue drafting another proposal.

Week 7: Implement the proposed solution and test as many edge cases as possible. Create guard functions to help prevent crashes and errors but try to limit the guard functions so that it doesn't create additional issues. Create a PR so that the maintainers can check my progress.

Week 8: Finalize the solution, implement suggested changes, and address any issues. Have the completed PR ready as soon as possible.

---

## Deliverables* 📦

*What will concretely exist at the end of the internship that does not exist today?*

The default canvas creation will be refactored so that it'll only be created when necessary. A canvas tracking boolean variable will exist to determine whether a user defined canvas or the default canvas will be created. Private guard and helper functions will be implemented to ensure that all p5 functions and addons are compatible with the refactored default canvas creation without any behaviors breaking. All p5.js operations will be unchanged. The main difference is that the canvas creation will be smarter and the architecture will be cleaner.

---

## Anything Else?
Our team would also like to continue working on issue #8922 the loading indicator issue, since the ones who started the implementation should finish the implementation. My teammate, @joshin, will be working on addressing the feedback for issue #8922 while I focus on this proposal if it is approved. 

*Anything you'd like maintainers to know that doesn't fit above.*

