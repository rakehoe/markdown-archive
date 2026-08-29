# Token-Efficient AI Prompt Template

A reusable prompt template for getting **clear, accurate, and consistent AI responses while minimizing unnecessary token usage**.

The goal is simple:

> **Give the AI only the information it needs to complete the task correctly.**

---

# 1. Minimal Prompt Template

Use this for simple tasks:

```text
TASK:
[What you want the AI to do]

CONTEXT:
[Only essential background information]

INPUT:
[Text, code, data, or requirements]

OUTPUT:
[Exactly what you want returned]

CONSTRAINTS:
[Important rules only]
```

### Example

```text
TASK:
Fix this React component.

CONTEXT:
React + Vite.

INPUT:
[paste code]

OUTPUT:
Return corrected code only.

CONSTRAINTS:
- Keep the existing structure.
- Don't add dependencies.
```

---

# 2. Recommended Compact Template

For most development tasks, use:

```text
ROLE: [expertise]

TASK: [specific objective]

CONTEXT: [essential context]

INPUT:
[code/data/content]

REQUIREMENTS:
- [requirement]
- [requirement]
- [requirement]

OUTPUT: [exact output format]
```

### Example

```text
ROLE: Senior React developer

TASK: Fix the routing.

CONTEXT: React + Vite + react-router-dom.

INPUT:
[paste App.jsx]

REQUIREMENTS:
- Keep existing pages.
- Use nested routes.
- Don't add dependencies.

OUTPUT: Return the corrected App.jsx only.
```

---

# 3. Ultra-Compact Template

When the task is straightforward, remove unnecessary sections:

```text
Fix [TASK] in [CONTEXT].

Input:
[INPUT]

Requirements:
- [RULE]
- [RULE]

Return:
[OUTPUT FORMAT]
```

Example:

```text
Fix the React Router configuration.

Context: React + Vite.

Input:
[paste code]

Requirements:
- Keep existing routes.
- Add /about.
- Add /contact.

Return: corrected App.jsx only.
```

---

# 4. Token-Saving Rules

## Rule 1 — State the Goal First

Bad:

```text
I am currently working on a React project and I have been
trying several things and I noticed that my routing isn't
working correctly...
```

Better:

```text
TASK: Fix React Router configuration.
```

The AI immediately knows the objective.

---

## Rule 2 — Remove Unnecessary Background

Don't explain information the AI doesn't need.

Instead of:

```text
I am a developer who has been learning React for several
months and I'm currently building a small website...
```

Use:

```text
CONTEXT: React + Vite.
```

---

## Rule 3 — Don't Repeat Information

Avoid:

```text
Use React.
This is a React project.
The application is built with React.
Please provide React code.
```

Use:

```text
CONTEXT: React + Vite.
```

---

# 5. Use Lists Instead of Long Explanations

Instead of:

```text
I want you to make sure that you don't change the existing
components and that you don't install any additional packages
and that you preserve the current functionality.
```

Use:

```text
REQUIREMENTS:
- Preserve existing components.
- No new dependencies.
- Preserve existing functionality.
```

Lists are easier for both humans and AI to process.

---

# 6. Specify the Output

One of the easiest ways to reduce unnecessary AI output is to explicitly define what you want returned.

Use:

```text
OUTPUT: Code only.
```

or:

```text
OUTPUT: One paragraph.
```

or:

```text
OUTPUT:
1. Diagnosis
2. Fix
3. Corrected code
```

or:

```text
OUTPUT: Return the complete README.md in Markdown.
```

---

# 7. Use "Only" When Appropriate

If you don't want explanations:

```text
OUTPUT: Return code only. No explanation.
```

If you want a short explanation:

```text
OUTPUT: Return the fix followed by a 2-sentence explanation.
```

If you want a specific file:

```text
OUTPUT: Return only the contents of App.jsx.
```

This prevents the AI from generating unnecessary content.

---

# 8. Separate Input from Instructions

Use clear boundaries:

```text
TASK: Fix the bug.

INPUT:
---START---
[paste code]
---END---

REQUIREMENTS:
- Preserve API.
- Don't change component names.

OUTPUT: Corrected code only.
```

This helps prevent the AI from confusing the provided content with instructions.

---

# 9. Code Prompt Template

Use this for programming tasks:

```text
ROLE: Senior [LANGUAGE/FRAMEWORK] developer

TASK: [specific coding task]

STACK: [framework/version/tools]

INPUT:
```[language]
[code]
```

REQUIREMENTS:
- [rule]
- [rule]
- [rule]

OUTPUT:
[exact format]
```

### Example

```text
ROLE: Senior React developer

TASK: Fix this component.

STACK: React + Vite + JavaScript

INPUT:
```jsx
[paste component]
```

REQUIREMENTS:
- Don't change the public API.
- Don't add dependencies.
- Preserve existing behavior.

OUTPUT: Corrected JSX only.
```

---

# 10. Debugging Prompt Template

```text
TASK: Debug this error.

ENVIRONMENT:
[language/framework/version]

ERROR:
[paste exact error]

CODE:
```text
[paste relevant code]
```

REQUIREMENTS:
- Identify the root cause.
- Give the smallest correct fix.
- Don't rewrite unrelated code.

OUTPUT:
1. Cause
2. Fix
3. Corrected code
```

---

# 11. Code Review Prompt Template

```text
ROLE: Senior [TECHNOLOGY] reviewer

TASK: Review the code below.

CODE:
```[language]
[code]
```

CHECK:
- Bugs
- Security
- Performance
- Maintainability

CONSTRAINTS:
- Don't rewrite unless necessary.
- Prioritize high-impact issues.

OUTPUT:
[severity] Issue → Fix
```

---

# 12. Refactoring Prompt Template

```text
TASK: Refactor this code.

INPUT:
```[language]
[code]
```

GOALS:
- [goal]
- [goal]

KEEP:
- Existing behavior
- Existing API

AVOID:
- New dependencies
- Unrelated changes

OUTPUT: Refactored code only.
```

---

# 13. Documentation Prompt Template

```text
ROLE: Technical writer

TASK: Create documentation for [PROJECT/FEATURE].

INPUT:
[code or notes]

INCLUDE:
- Purpose
- Installation
- Usage
- Examples

STYLE:
- Concise
- Developer-friendly
- Scannable

OUTPUT: Markdown only.
```

---

# 14. README Prompt Template

```text
ROLE: Senior technical writer

TASK: Create a README.md for [PROJECT].

CONTEXT:
[project description]

INCLUDE:
- Overview
- Requirements
- Installation
- Usage
- Project structure
- Configuration
- Examples
- Troubleshooting

STYLE:
- Concise
- Practical
- Scannable

OUTPUT: Complete README.md only.
```

---

# 15. Writing / Rewriting Prompt Template

```text
TASK: [rewrite/summarize/proofread/etc.]

INPUT:
---START---
[text]
---END---

GOAL:
[desired result]

STYLE:
[tone]

CONSTRAINTS:
- Preserve meaning.
- Remove unnecessary words.
- [additional rule]

OUTPUT: Final version only.
```

---

# 16. Data Analysis Prompt Template

```text
ROLE: Data analyst

TASK: [specific analysis]

DATA:
[paste data]

QUESTIONS:
1. [question]
2. [question]

RULES:
- Use only supplied data.
- State assumptions.
- Don't invent missing values.

OUTPUT:
[table/summary/calculation/etc.]
```

---

# 17. Multi-Step Task Template

For more complex tasks, use numbered steps:

```text
ROLE: [role]

TASK:
Complete these steps:

1. [step]
2. [step]
3. [step]

CONTEXT:
[essential context]

INPUT:
[input]

CONSTRAINTS:
- [constraint]
- [constraint]

OUTPUT:
[expected final result]
```

Keep the steps short.

---

# 18. "Do Not" Rules

Use only important negative constraints.

Good:

```text
DO NOT:
- Add dependencies.
- Change the API.
- Modify unrelated files.
```

Avoid excessive restrictions such as:

```text
Don't do this.
Don't do that.
Don't explain this.
Don't change that.
Don't use this.
Don't use that.
Don't...
```

Too many negative instructions make prompts longer and can make the task harder to follow.

---

# 19. Ask for the Smallest Correct Change

For coding tasks, this is especially useful:

```text
REQUIREMENT:
Make the smallest change necessary to fix the issue.
```

Or:

```text
PRIORITY:
Correctness > minimal changes > style improvements.
```

This discourages unnecessary rewrites.

---

# 20. Avoid Asking for Repetition

Don't write:

```text
First explain the problem.
Then explain what you're going to do.
Then explain the solution.
Then provide the solution.
Then summarize the solution.
```

Instead:

```text
OUTPUT:
Cause + fix + corrected code.
```

---

# 21. Use Variables for Reusable Prompts

For prompts you use repeatedly:

```text
ROLE: {ROLE}

TASK: {TASK}

CONTEXT: {CONTEXT}

INPUT:
{INPUT}

REQUIREMENTS:
- {REQUIREMENT_1}
- {REQUIREMENT_2}

OUTPUT: {OUTPUT}
```

Example:

```text
ROLE: Senior React developer

TASK: {TASK}

STACK: React + Vite

INPUT:
{CODE}

REQUIREMENTS:
- Preserve behavior.
- No new dependencies.

OUTPUT: {FORMAT}
```

---

# 22. Prompt Priority

When instructions conflict, make priorities explicit:

```text
PRIORITY:
1. Correctness
2. Existing functionality
3. Minimal changes
4. Readability
```

This is especially useful for code modification tasks.

---

# 23. Context Budget

Only include context that can change the answer.

### Include

```text
Framework: React
Version: 19
Language: JavaScript
Problem: Route returns 404
```

### Usually exclude

```text
I started this project last week.
I'm learning React.
I use VS Code.
I really want the code to be clean.
```

unless those facts actually affect the requested solution.

---

# 24. Compact Prompt Formula

A good general-purpose formula is:

```text
ROLE + TASK + CONTEXT + INPUT + CONSTRAINTS + OUTPUT
```

For example:

```text
ROLE: Senior React developer

TASK: Fix the routing bug.

CONTEXT: React + Vite + React Router.

INPUT:
[paste code]

CONSTRAINTS:
- Minimal changes.
- No new dependencies.

OUTPUT: Corrected App.jsx only.
```

---

# 25. One-Line Prompt Formula

For very simple requests:

```text
[Task] + [Context] + [Constraint] + [Output]
```

Example:

```text
Fix this React component. React + Vite. No new dependencies. Return corrected code only.
```

---

# 26. Token-Efficient Prompt Checklist

Before sending a prompt, ask:

- [ ] Is the task stated clearly?
- [ ] Did I remove unnecessary background?
- [ ] Did I avoid repeating information?
- [ ] Did I provide only relevant context?
- [ ] Are requirements short and explicit?
- [ ] Did I specify the desired output?
- [ ] Did I separate input from instructions?
- [ ] Did I remove unnecessary examples?
- [ ] Did I avoid redundant explanations?
- [ ] Did I specify important constraints only?

---

# 27. Best Default Template

When unsure which template to use, start here:

```text
ROLE: [expertise]

TASK: [specific objective]

CONTEXT: [essential information]

INPUT:
[relevant content]

REQUIREMENTS:
- [must-have]
- [must-have]
- [important constraint]

OUTPUT: [exact format]
```

## Example

```text
ROLE: Senior frontend developer

TASK: Fix the navigation.

CONTEXT: React + Vite + React Router.

INPUT:
[paste relevant code]

REQUIREMENTS:
- Preserve existing routes.
- No new dependencies.
- Make the smallest correct change.

OUTPUT: Corrected code only.
```

---

# 28. Golden Rule

> **Don't make the prompt longer to make it clearer. Make it more structured.**

A token-efficient prompt should answer six questions:

```text
WHO?      → ROLE
WHAT?     → TASK
CONTEXT?  → CONTEXT
WITH WHAT?→ INPUT
RULES?    → REQUIREMENTS
RESULT?   → OUTPUT
```

The ideal prompt is not the shortest possible prompt.

It is the **shortest prompt that removes ambiguity**.