---
title: Language Plasticity is More Important Than Ever
author: Adam Wilson

categories:
  - Machine Learning
  - Community
  - Guest Posts
  - Tutorials
---

Large language models are rewriting how we build software. From quick prototypes to full features, prompting an LLM to generate or refactor code in the D programming language feels increasingly natural. But as context windows grow and API bills mount, a subtler question emerges: does the language itself influence how many tokens an LLM consumes and therefore how much it costs to work with?

D, with its blend of systems-level power and modern conveniences, offers us an intriguing lens through which to evaluate LLM Token usage. While the community often champions D for performance and expressiveness, its static typing, consistent syntax, and thoughtful design choices might also deliver hidden efficiencies when paired with AI tools. Let's explore three angles where language design intersects with token economics, with a light nod to the ongoing debates that keep us all entertained.

## Tabs, Spaces, and Silent Token Costs

The tabs-versus-spaces debate has long divided developers. In the LLM era, it carries real financial weight. Many tokenizers treat whitespace as distinct tokens, and common conventions amplify the difference.

Python's official style guide recommends four spaces for indentation. In practice, this means every level of nesting can add multiple individual space tokens. D, like C-family languages, relies on braces for structure, so indentation serves readability rather than syntax. This flexibility lets teams choose tabs (a single character) without breaking semantics.

A recent study on code formatting and LLM budgets quantified the impact. Removing formatting elements like indentation, extra whitespace, and newlines, reduced input tokens by an average of 24.5% across models and languages, with negligible effects on output quality (Pass@1 drops under 4.2% on average). Java saw the largest savings (~35%), while Python's reliance on significant whitespace limited reductions to around 6%. Indentation alone often accounted for 8–10% of tokens in C-style languages.

For a quick case study, consider equivalent snippets. A deeply nested Python function with four-space indents piles on tokens quickly. The same logic in D, using braces and tabs, trims the count, especially if your formatter or LLM prompt favors compact style. The takeaway? Consistent, minimal formatting isn't just aesthetic; it's a budget hack. Many teams are now experimenting with unformatted prompts for generation, then pretty-print afterward.

## Duck Typing vs. Explicit Context

Dynamic languages shine for rapid iteration, but their flexibility can demand extra tokens from the LLM. In duck-typed environments like Python or JavaScript, a variable's behavior emerges only through usage. When an LLM builds context, it often needs surrounding code, comments, or explicit hints to infer "this variable quacks like a list here, but might be something else later."

Statically typed languages flip the script. In D, a parameter declared as {% raw %}`int[] arr`{% endraw %} or a struct with explicit methods tells the model exactly what operations are valid, right there in the signature. No detective tokens required.

The result? When using LLMs with duck-typed languages, the model often spends more tokens reconstructing intent. Explicit types and consistent call-site semantics in D let it focus on logic instead of inference.

## Features That Make Refactoring LLM-Friendly

Language design choices ripple through edit tasks. One classic pain point in C++ is the distinction between {% raw %}`.`{% endraw %} for value types and {% raw %}`->`{% endraw %} for pointers/references. Change a type from {% raw %}`Foo`{% endraw %} to {% raw %}`Foo*`{% endraw %}, and suddenly every member access across the codebase flips operators. LLMs tasked with this refactor must hunt down and update dozens, or hundreds, of call-sites, burning tokens on repetitive churn.

D sidesteps this entirely. Member access uses the dot operator uniformly for structs, classes, and even pointers to them. No {% raw %}`->`{% endraw %} operator exists for this purpose. Refactoring a value type to a reference (or vice versa) often requires only changing the declaration and a few dereferences, far less widespread editing.

D brings additional plasticity through Uniform Function Call Syntax (UFCS). Any free function can be called as a method: {% raw %}`foo(bar)`{% endraw %} becomes {% raw %}`bar.foo()`{% endraw %}. This enables seamless chaining and lets you extend types without modifying their definitions. When an LLM suggests adding a helper, you can often integrate it via UFCS without touching call sites or introducing new boilerplate.

Other D features amplify this. Compile-time function execution (CTFE) and powerful templates let the compiler handle complexity that might otherwise require verbose runtime code in other languages. When prompting for changes, the model can lean on these abstractions rather than generating repetitive patterns.

## Weighing the Evidence

Data from token-efficiency analyses show dynamic languages often win on raw count: no type annotations means fewer tokens per line. One comparison across RosettaCode tasks found a 2.6× gap between the least efficient (C) and most efficient languages, with dynamic options and strongly-inferring functional languages (Haskell, F#) leading the pack. JavaScript proved surprisingly verbose among dynamics, while pure static languages paid a clear price for declarations.

Yet the picture isn't purely about fewer tokens. Static typing provides immediate, machine-checkable context that can reduce the LLM's need for explanatory scaffolding. Combined with D's consistent operators, UFCS, and brace-based structure, this often translates to lower churn during iterative edits, the very tasks that dominate real-world LLM usage.

Ultimately, token efficiency is one factor among many. D's compile-time feedback, memory safety options, and expressiveness pair well with rapid AI iteration. Whether you land on "static wins on cost" or "dynamic saves so much time" depends on your workload. The real experiment? Try prompting the same task in D versus Python or C++, measure the tokens, and see what your budget (and your code) tells you.

What are your experiences prompting LLMs in D? Drop a comment in the forums. I'm genuinely curious how the community is navigating this new landscape.

---

*References and further reading:*
- "How Code Formatting Silently Consumes Your LLM Budget" (arXiv): https://arxiv.org/html/2508.13666v1
- "Which programming languages are most token-efficient?" (martinalderson.com): https://martinalderson.com/posts/which-programming-languages-are-most-token-efficient/
- D Language Tour – Uniform Function Call Syntax: https://tour.dlang.org/tour/en/gems/uniform-function-call-syntax-ufcs
- D forums discussion on struct/class member access: https://forum.dlang.org/thread/mbwkdoomvqhhtdidyxce@forum.dlang.org

Happy coding—and may your token counts be ever in your favor!
