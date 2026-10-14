# SplitIt

A small app for splitting costs inside a group (a trip, a flatshare, a dinner series) and
working out who should transfer what to whom to clear the balances. It produces
instructions only; it never moves money.

It is also a deliberate vehicle for learning **event modeling** and **event sourcing**.
Domain state is event-sourced; identity (users, sign-in codes, keys) is not.

## Tech stack

- C# on .NET 10, ASP.NET Core with Razor components for the UI
- [Marten](https://martendb.io) on PostgreSQL for the event store and projections
- [Wolverine](https://wolverine.netlify.app) for HTTP endpoints and command handling
- ASP.NET Core Identity on EF Core (Npgsql) for sign-in
- OpenTelemetry for tracing and metrics
- Docker Compose for the local database; the container image is built by `Dockerfile`

## Architecture

The app is built as independent vertical slices, one per step of the event model, under
`src/SplitIt/Slices/`. A slice holds its own events, state, decider, endpoint and page.
Duplication between slices is preferred over shared logic. Cross-cutting code lives in
`src/SplitIt/Infrastructure/` and `src/SplitIt/Shared/`.

Tests live in `tests/SplitIt.Tests`. They seed events directly and drive only their own
slice's screen.

## Design documents

- [`spec.md`](spec.md): scope, decisions and non-goals.
- [`docs/event-model/event-model.yaml`](docs/event-model/event-model.yaml): the source of truth for the event model (screens, commands, events, read models).
- `docs/event-model/slice-*.md`: one doc per slice, with its Given/When/Then specifications.
- `docs/event-model/happy-path.png`: diagram generated from the YAML.

Work is design-first: update the event model, the slice doc and `spec.md`, get them
reviewed, then implement.

## Getting started

Prerequisites: the .NET SDK (version in `global.json`), Docker, [`just`](https://just.systems), [`d2`](https://d2lang.com) and Python 3.

```sh
just setup   # dotnet tools, Python venv for the model generator, HTTPS dev certificate
just run     # starts Postgres, runs the app on https://localhost:7194, opens sign-in
just test    # run the tests
```

## The justfile

All commands go through `just`; run `just` to list them. The main ones:

| Recipe | Purpose |
|---|---|
| `build` / `test` | Build (warnings are errors) and run tests; `just test --filter Name` narrows |
| `run` / `watch` | Run the app, or run it with live reload |
| `model` / `model-check` | Validate `event-model.yaml` and render the diagram, or validate only |
| `db` / `db-stop` / `psql` / `logs` | Manage and inspect the local Postgres |
| `events` | Print the event stream |
| `rebuild-projections` | Rebuild stored projections after changing a fold |
| `reset-ledger` | Drop all domain data (dev only) |
| `migration <name>` | Add an EF Core migration for Identity |
| `describe` / `projections` | Inspect discovered endpoints and projections |

## Contributing

See [`AGENTS.md`](AGENTS.md) for the working conventions: design before code, refer to
slices by name, keep comments to a minimum, and keep slices independent. Scratch files go in `tmp/`.
