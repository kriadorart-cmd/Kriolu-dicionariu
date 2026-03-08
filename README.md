# Kriolu dicionáriu

## Current status

This repository is currently a bootstrap repository: it has Git history but no application code yet.

## Newcomer guide

### What exists today

- Git repository initialized with a `work` branch.
- No source directories (`src/`, `app/`, etc.) yet.
- No build, test, or CI configuration yet.

### Suggested project structure to adopt

A practical starting structure for a dictionary-style application:

```text
.
├── README.md
├── docs/
│   ├── architecture.md
│   ├── api.md
│   └── contributing.md
├── src/
│   ├── api/
│   ├── domain/
│   ├── services/
│   └── storage/
├── tests/
│   ├── unit/
│   └── integration/
└── scripts/
```

### Important decisions to make early

1. **Primary runtime and framework** (e.g., Python/FastAPI, Node/NestJS, etc.).
2. **Data model for entries** (lemma, part of speech, variants, examples, metadata).
3. **Search strategy** (exact match, prefix search, fuzzy match, normalization rules).
4. **Data storage** (flat files for early stage vs SQL/NoSQL for scale).
5. **Quality checks** (formatter, linter, test runner, CI commands).

### What to learn next

- Define a one-feature vertical slice: “lookup term by headword”.
- Add tests first for the lookup behavior and normalization edge-cases.
- Document contribution flow and coding standards in `docs/contributing.md`.
- Add an architecture note describing boundaries between API/domain/storage.

## First milestones

1. Add the chosen runtime scaffold and lockfile.
2. Implement `GET /entries/{term}` (or CLI equivalent).
3. Add initial fixture dataset with 20–50 entries.
4. Add unit tests and one integration test.
5. Add CI command parity with local development.
