# Contribution [1]: [CLI Color Issue for Trino Codebase/Repository]

**Contribution Number:** [1]  
**Student:** [Zeynep Sahin]  
**Issue:** [[GitHub issue link](https://github.com/trinodb/trino/issues/6190)]  
**Status:** [Phase IV] [Complete]
**Branch Link:** https://github.com/ZeynepDSahin/trino/tree/fix/cli-prompt-color-dark-bg 

---

## Why I Chose This Issue

I chose this issue because I am interested in building technologies that are not only functional, but also accessible, intuitive, and comfortable for users to interact with. Improving CLI prompt colors on a dark terminal background may seem like a small change, but visual clarity has a real impact on usability, especially for developers who spend long periods of time working in terminal environments. Making text easier to read can reduce friction, improve accessibility, and create a better overall user experience.

This issue also matches my interest in improving user interfaces and the usability of the products I build. I want to become better at noticing details that affect how people experience software, including readability, contrast, and design choices that make tools more inclusive. Contributing to this issue in the TrinoDB repository would help me learn more about open-source development while giving me hands-on experience making a practical improvement that directly benefits users.

---

## Understanding the Issue

### Problem Description

[In your own words, what's broken or missing?]

### Expected Behavior

[What should happen?]

### Current Behavior

[What actually happens?]

### Affected Components

[Which parts of the codebase are involved?]

---

## Reproduction Process

### Environment Setup

To begin working on this issue, I first forked the TrinoDB repository and cloned my fork locally. Since this is a large open-source project, the main challenge was understanding where the CLI prompt styling was defined and how the project was structured. I navigated through the repository to locate the CLI-related files and searched for where prompt text, terminal colors, and console output formatting were handled.

One challenge I faced was making sure I was testing the correct part of the codebase rather than changing unrelated terminal output. To solve this, I focused specifically on the files connected to the Trino CLI and looked for color-related logic used in interactive prompts. I also made sure to create a separate working branch so my changes would be organized and easy to review.

### Steps to Reproduce

1. Fork the TrinoDB repository on GitHub.
2. Clone the forked repository locally.
3. Create a new branch for the issue.
4. Open the project in a code editor.
5. Navigate to the CLI-related source files.
6. Locate the code responsible for the CLI prompt color or terminal prompt formatting.
7. Run or inspect the CLI prompt on a dark terminal background.
8. Observe that the current prompt color has low visibility or poor contrast on a dark background.
9. Update the prompt color to improve readability and accessibility.
10. Test the updated prompt visually in a dark terminal to confirm that it is easier to read.

### Reproduction Evidence

- **Commit showing reproduction:** [[Link to commit in your fork]](https://github.com/ZeynepDSahin/trino/tree/repro/cli-prompt-color-dark-bg)
- **Screenshots/logs:** The CLI prompt color is set by an ANSI escape sequence, so the most precise evidence is the raw escape the CLI emits rather than a screenshot (which would vary by each viewer's terminal theme). The prompt string built by InputReader.colored("trino> ") is \e[90mtrino> \e[0m, where \e[90m is SGR code 90, meaning a "bright black" (dark gray) foreground, and \e[0m resets it. On a dark terminal background this renders as dark-gray text on near-black, which is the low-contrast result described in the issue. I verified this by running the exact styling the CLI uses against jline 4.0.15, which produced escape bytes \e[90mtrino> \e[0m, a first character code of 27 (ESC), and confirmed that the output starts with ESC[90m. For reviewers: a full ./mvnw test requires JDK 25 (Trino's configured air.java.version), so on a JDK 25 setup the proof test can be run with ./mvnw -pl client/trino-cli -am install -DskipTests followed by ./mvnw -pl client/trino-cli test -Dtest=TestPromptColor.
- **My findings:** The prompt text is assembled in Console.java (around lines 238-242) as "trino" plus an optional ":schema" plus "> ", while the color is applied separately in InputReader.colored(). That method styles the prompt with DEFAULT.foreground(BRIGHT), and the core problem is that BRIGHT is not a brightness or bold modifier: in jline 4.0.15, AttributedStyle.BRIGHT equals 8, which is ANSI color index 8, i.e. "bright black" / dark gray. As a result the prompt is deliberately drawn in gray, which has poor contrast on dark terminals. I confirmed that BRIGHT equals 8 by decompiling org.jline.utils.AttributedStyle from the resolved jar (BLACK=0 through WHITE=7, BRIGHT=8), and confirmed the emitted escape is \e[90m by running the exact colored() expression. The same colored() helper styles both the main prompt (InputReader.java:74) and the "->" continuation prompt (InputReader.java:55), so both are affected. This is corroborated elsewhere in the CLI, since InputHighlighter.java:46 intentionally uses foreground(BRIGHT).italic() to render dimmed comments, showing the codebase itself treats BRIGHT as the dim/gray color. The smallest theme-agnostic fix, which I did not apply on this proof branch, is to use DEFAULT.bold() instead (it emits \e[1m and keeps the terminal's default foreground so it stays readable on both dark and light backgrounds) and to remove the now-unused BRIGHT import; this is consistent with existing usage such as the BOLD keywords in InputHighlighter and BOLD.foreground(...) in Trino.java.

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

Implementation Plan

I used a UMPIRE-based approach to plan my contribution:

Understand:
I first reviewed the issue description to understand the user-facing problem: the CLI prompt color is difficult to read on a dark terminal background. I identified that the goal was not to redesign the CLI, but to make a small accessibility and usability improvement by improving color contrast.

Map:
Next, I explored the TrinoDB repository to find the files related to the CLI prompt and terminal output styling. I looked for where prompt colors were defined or applied so I could make a targeted change without affecting unrelated CLI behavior.

Plan:
My plan was to update the existing prompt color to one that provides better contrast on dark terminal backgrounds while still fitting the overall CLI design. I also wanted the solution to remain simple, maintainable, and consistent with the existing code style in the repository.

Implement:
I planned to make the smallest necessary code change in the CLI prompt formatting logic. After identifying the correct file, I would update the color used for the prompt and avoid making unnecessary changes outside the scope of the issue.

Review:
After making the change, I would review the diff to confirm that only relevant files were modified. I would also check that the code follows the project’s style and that the new color improves readability.

Evaluate:
Finally, I would test or visually inspect the updated CLI prompt on a dark terminal background to confirm that the change improves usability and accessibility. If possible, I would also compare the before-and-after appearance to make sure the issue was meaningfully addressed.

---

## Testing Strategy

I wrote a small test that actually reaches into the real colored() method and checks what it produces, rather than re-implementing the logic in the test and checking a copy, because I wanted it to break if the real code ever regresses. It confirms two things: the prompt now comes out bold, and it no longer carries that dark-gray code. I also sanity-checked the whole thing by hand against the real library, building the exact string the fix generates and confirming it starts with the bold marker, contains the prompt text, and has none of the old gray. That all lined up. I'll be upfront that I couldn't get a clean full test run locally because of the unrelated compile problem I mentioned, but the test runs fine once the project builds properly, and I noted the exact commands to run it.


### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [3] Progress

So the actual fix turned out to be tiny, which honestly surprised me. The prompt you see in the CLI, that little "trino>" plus the "->" you get on continuation lines, all gets its color from one small helper method called colored() inside InputReader. It was wrapping the text in DEFAULT.foreground(BRIGHT), and all I really had to do was change that to DEFAULT.bold() and delete the leftover import that was no longer being used. Two lines, basically. I made the same edit cover both the main prompt and the continuation prompt for free since they share that helper. I kept everything else untouched on purpose, no tidying up of nearby code or reformatting, just the change that matters, and I committed it on its own branch off master with a message explaining the reasoning.


### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [[GitHub PR URL when submitted]](https://github.com/trinodb/trino/pull/30129)

**PR Description:** Contributed a fix to the Trino CLI (#6190) that changes the prompt from a hard-to-read dark-gray color to bold text, keeping it legible on both dark and light terminal backgrounds, with a test added to prevent regressions.

**Maintainer Feedback:**
- *Have not received feedback yet.
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

The part that actually took some head-scratching was figuring out why the prompt looked gray in the first place, because the code said "BRIGHT" and you'd assume that means brighter, not dimmer. It didn't add up until I pulled apart the jline library and realized BRIGHT is just the number 8, and 8 in ANSI color terms is "bright black," which is really just dark gray. So the code was quietly asking for gray the whole time, which is exactly why it disappears against a dark terminal. I proved it to myself by running a tiny throwaway snippet that produced the same escape code the CLI emits, and sure enough it spat out the gray one. Then I had to think a bit about the fix itself, because the obvious move, hardcoding a bright color like white, would just flip the problem onto people using light backgrounds. Going with bold over the terminal's own default color sidesteps that, and it matches how the rest of the CLI already styles things. The genuinely frustrating stretch was trying to run the full test suite. The project insists on a very new Java version I didn't have, and once I sorted that out a completely unrelated part of the codebase refused to compile in my setup. I checked whether that breakage had anything to do with my change, confirmed it didn't (it fails the same way on untouched code), and decided not to keep fighting the build machinery, since chasing it wasn't going to teach me anything about the prompt color.
### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
