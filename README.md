# Contribution [#]: [Issue Title]

**Contribution Number:** [1]  
**Student:** [Zeynep Sahin]  
**Issue:** [[GitHub issue link](https://github.com/trinodb/trino/issues/6190)]  
**Status:** [Phase II] [Complete]

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

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
