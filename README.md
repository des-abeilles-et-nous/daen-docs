# daen platform documentation

Single source of truth for ecosystem-level information across the DAEN-SCOUT (Beefree) platform: architecture, data model, environments, and development framework.

For repo-specific build/dev guidance, see each repo's own `CLAUDE.md` ([daen-scout](https://github.com/des-abeilles-et-nous/daen-scout), [daen-fb-workers](https://github.com/des-abeilles-et-nous/daen-fb-workers), [fb-admin](https://github.com/des-abeilles-et-nous/fb-admin)).

## Start here

- **[ECOSYSTEM_CONTEXT.md](ECOSYSTEM_CONTEXT.md)** — high-level architecture, environments, naming standards, Firebase data structures, commit conventions. Required reading; referenced from every repo's `CLAUDE.md`.
- **[CLAUDE.md](CLAUDE.md)** — guide for AI agents working on this docs repo (factorization principle, contribution rules).

## High Level Design

![](https://github.com/des-abeilles-et-nous/daen-docs/blob/main/environments/beefree%20HLD.drawio.svg)

## Contents

| Section | What's there |
|---|---|
| [`data model/`](data%20model/) | [POI lifecycle](data%20model/POI%20lifecycle.md), [User model](data%20model/User%20model.md), [Subscriptions](data%20model/Subscriptions.md), [Feeds](data%20model/Feeds.md), [daen-scout MDD](data%20model/mdd%20daen-scout.md) |
| [`data pipelines/`](data%20pipelines/) | Data Warehouse setup, ETL diagrams |
| [`backend/`](backend/) | Worker System reference |
| [`dev framework/`](dev%20framework/) | React Native conventions, Bit component management, ICOMOON |
| [`environments/`](environments/) | Environments management, HLD diagrams, system landscape |
| [`testing/`](testing/) | Test suite master table, suite definitions |

## Documentation factorization

Ecosystem-level information lives **only here**, never duplicated in individual repos. Repo `CLAUDE.md` files reference back via `../daen-docs/...` (local) or the GitHub URL (single-repo clones). If you spot an ecosystem-level topic duplicated inside a repo, replace it with a link.

## Contributing

1. Place content in the right top-level section above.
2. Use Markdown; link to code with relative paths; include examples and diagrams where useful.
3. Update this README's table when you add a new top-level section or a notable file.
4. Get review before merging significant changes — out-of-date docs are worse than missing ones.

## License

See [LICENSE](LICENSE).
