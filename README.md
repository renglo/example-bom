# example-bom

Template for a new tenant **`*-bom`** repository (apollo-bom, acme-bom, …).

Copy this tree into a new GitHub repo, rename it, then fill in real pins and tenants.
Deploy scripts live in **[renglo/bom-helper](https://github.com/renglo/bom-helper)** — this repo only holds system-specific config.

## Create a new system BOM

```bash
# from a Stanley-style workspace
cp -R ops/example-bom ops/apollo-bom
cd ops/apollo-bom
# point origin at the new empty GitHub repo, then:
```

1. Edit `deploy_targets.yml`
   - Set `bom:` / optional `handlers_bom:`
   - Set `helper.ref` (pin a bom-helper tag when you cut one; `main` is fine while bootstrapping)
   - Replace the `example` tenant with your AWS account / id / stages
2. Edit `bom/v0.1.0.json` (or copy forward to `v0.1.1`, …) with real `python` / `npm` / `repos` pins
3. Commit and push to `main` (after launcher OIDC trusts this GitHub repo)

## Files you own

| File | Role |
|------|------|
| `bom/vX.Y.Z.json` | Backend + console package / git pins |
| `handlers_bom/vX.Y.Z.json` | Optional external handlers pins |
| `deploy_targets.yml` | Which BOM versions deploy, registry, tenants, **helper pin** |
| `.github/workflows/*` | Thin wrappers that check out bom-helper |

Do **not** copy `scripts/` or `Dockerfile` into this repo. CI runs:

```yaml
- uses: actions/checkout@v4
- uses: renglo/bom-helper/.github/actions/use-helper@main
```

## git-convoy

Unchanged:

```bash
git convoy adopt --bom ops/<system>-bom
# commit + push the *-bom repo → CI deploys
```

## Local checks

With bom-helper checked out as a sibling:

```bash
python3 ../bom-helper/scripts/bom_manifest.py --plan --pipeline backend bom/v0.1.0.json
python3 ../bom-helper/scripts/bom_manifest.py --validate --pipeline console bom/v0.1.0.json
```
