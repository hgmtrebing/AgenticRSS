## Overview

This project is to create an RSS Aggregator Server, which aggregates RSS feeds and makes them available for RSS Clients. Some essential features:
    - API so that AI Agents can read and write from the service. This will become more important as development continues - AI will ultimately be a critical component on top of the vanilla RSS Server.
    - Ability to create RSS feeds by scraping websites
    - UI to administer the server and view RSS feeds
    - API so that other RSS clients can read from the server

## Technology Stack

The backend server will be coded in Java / Spring Boot. The attached database will be Postgres. The UI should be coded in React and served from the database. The API should be coded in Swagger / Open API.

## Coding Style

- Favor smaller, single-purpose methods and functions. A soft guideline is fifty lines or less. It is acceptable if a method or function only has a single caller. The intention is to leverage functions as named blocks of code, to improve readability and comprehensibility.
- When writing comments, favor WHY comments over HOW comments. Comments should describe why something exists or the purpose it serves, rather than how it functions. Ideally, the code itself should document the precise implementation.
- Write JavaDoc-style comments (or whatever the language's equivalent is) for all methods and classes. Method comments can be brief, class comments can be longer.
- Put comments on non-trivial branching statements (if, else, while, for, switch, case, etc.). These comments can be brief, but should describe WHY the branch exists and what it is trying to accomplish. 
- Aim to keep classes and files on the small side. A soft target of 1000 lines or less is ideal.
- Favor verbose, self-documenting naming conventions. Avoid unclear abbreviations.
