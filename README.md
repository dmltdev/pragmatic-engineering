# Pragmatic Engineering

Portable Agent Skills for product-focused engineering decisions and evidence-backed quality gates.

## Lenses

Change-safe engineering and engineering gates are complementary views over the same skill: one guides the smallest maintainable change, while the other requires evidence that the selected boundary works. They are not separate directories or products.

## Skills

| Skill | Kind | Surface | Lens | Trigger |
|---|---|---|---|---|
| `dependency-seam` | Case discipline | Dependency boundaries | Change-safe engineering + engineering gates | Core logic meets app/provider/foreign semantics, or an adapter, DI, replaceability, or deletability decision is requested. |

## Install

### skills.sh

```bash
npx skills add dmltdev/pragmatic-engineering --skill dependency-seam
```

### Local source

```bash
git clone https://github.com/dmltdev/pragmatic-engineering.git
npx skills add ./pragmatic-engineering --skill dependency-seam
```

## Responsibility Boundary

This package owns reusable engineering disciplines, principles, references, and evidence gates, while product discovery/acceptance, lifecycle state, personas, execution profiles, git/forge/deployment/session tooling, and host packaging remain elsewhere.

## Clean-Cutover Capability

When existing code already leaks provider semantics, `dependency-seam` decides the destination and hands off the behavior-preserving migration as a clean-cutover capability. Void Grimoire’s `refactor-transaction` is the current optional provider; it is not a dependency of this repository and is not invoked automatically.

## License

MIT. See [LICENSE](LICENSE).
