# Project Context

## Usage Guide

This file contains the context of the application under test. **The agent should read this file before starting any automation task** to correctly understand the domain and tech stack.

> ⚠️ **You need to update this file** for each specific project. Below is a template.

---

## Application Overview

- **Application name:** [App name]
- **Description:** [Short description of the application]
- **Type:** [Web App / Mobile App / API]
- **Test environment URL:** [Staging/test URL]

## Tech Stack

- **Frontend:** [React / Angular / Vue / ...]
- **Backend:** [Java Spring / Node.js / .NET / ...]
- **Database:** [MySQL / PostgreSQL / MongoDB / ...]
- **Authentication:** [JWT / OAuth2 / Session-based / ...]

## Key Features & Modules

| Module          | Description                 | Priority |
| --------------- | --------------------------- | -------- |
| Login           | Login, forgot password, 2FA | High     |
| Dashboard       | Overview page               | Medium   |
| User Management | CRUD users, permissions     | High     |
| ...             | ...                         | ...      |

## Environment Details

| Environment | URL | Credentials               |
| ----------- | --- | ------------------------- |
| Dev         | ... | ...                       |
| Staging     | ... | ...                       |
| Production  | ... | N/A (do not test on prod) |

## Notes

- Add any special notes about business rules, edge cases, or known issues here.
