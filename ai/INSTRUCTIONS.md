
## Coding Style

- Favor smaller, single-purpose methods and functions. A soft guideline is fifty lines or less. It is acceptable if a method or function only has a single caller. The intention is to leverage functions as named blocks of code, to improve readability and comprehensibility.
- When writing comments, favor WHY comments over HOW comments. Comments should describe why something exists or the purpose it serves, rather than how it functions. Ideally, the code itself should document the precise implementation.
- Write JavaDoc-style comments (or whatever the language's equivalent is) for all methods and classes. Method comments can be brief, class comments can be longer.
- Put comments on non-trivial branching statements (if, else, while, for, switch, case, etc.). These comments can be brief, but should describe WHY the branch exists and what it is trying to accomplish. 
- Aim to keep classes and files on the small side. A soft target of 1000 lines or less is ideal.
- Favor verbose, self-documenting naming conventions. Avoid unclear abbreviations.
