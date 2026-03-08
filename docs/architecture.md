# Architecture (starter draft)

This draft captures a newcomer-friendly baseline architecture for Kriolu dicionáriu.

## Layers

- **API layer**: Handles request/response and validation.
- **Domain layer**: Dictionary concepts and business rules.
- **Service layer**: Use-cases such as lookup, autocomplete, related forms.
- **Storage layer**: Reads/writes from chosen persistence backend.

## Principles

1. Keep dictionary rules in the domain layer, not controllers.
2. Keep storage swappable via interfaces.
3. Prefer deterministic normalization functions for search.
4. Keep fixtures small and readable for early tests.

## First use-case to implement

- Input: raw search term.
- Process: normalize -> lookup -> rank candidates.
- Output: structured entry payload (definitions, examples, variants).

## Risks to manage early

- Inconsistent orthography handling.
- Accent/diacritic normalization errors.
- Tight coupling between API and storage models.
