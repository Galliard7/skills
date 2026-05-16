---
name: grill-me
description: Interview the user relentlessly about a plan or design until reaching shared understanding, resolving each branch of the decision tree.
---

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one.

## How to ask questions

Use the AskUserQuestion tool for every question. Present each question as multiple choice:

- Provide 3-5 options labeled A, B, C, etc.
- One option should be your recommended answer — mark it with "(recommended)" next to the label.
- Always include a final option: "Other (I'll explain)"
- Add a one-sentence rationale for your recommended option in the question body.
- Wait for the user's response before moving to the next question.

If a question can be answered by exploring the codebase, explore the codebase instead of asking.
