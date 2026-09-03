# example-bom

Public **GitHub template** for a tenant deploy repo named `<tenant>-bom`
(for example `apollo-bom`, `acme-bom`).

This repository holds **only system-specific config**: which package versions
to install and which AWS tenants to deploy to. Shared deploy scripts live in
**[renglo/bom-helper](https://github.com/renglo/bom-helper)** and are checked
out automatically by GitHub Actions — you do not copy those scripts into your
BOM repo.

---

## Create a new `<tenant>-bom` repository

You need a GitHub account that can create repositories in the **destination
organization** (the org that will own the new BOM, for example `mycompany`).

### 1. Create the repo from this template

1. Open **[renglo/example-bom](https://github.com/renglo/example-bom)** in the browser.
2. Click **Use this template** → **Create a new repository**.
3. Choose:
   - **Owner:** the destination organization (or your user).
   - **Repository name:** `<tenant>-bom` (example: `apollo-bom`).
   - **Visibility:** usually **Private** for production systems.
4. Click **Create repository**.

GitHub copies every file from this template into the new repo. You do **not**
need to `git clone` `example-bom` first, and you do **not** need any other
local project layout.

> Creating a repo **in another organization** from a public template is
> supported as long as your account has permission to create repositories in
> that org. If the Owner dropdown does not list the org, ask an org admin to
> invite you (or create the repo for you).

### 2. Clone your new repo locally

```bash
git clone git@github.com:<ORG>/<tenant>-bom.git
cd <tenant>-bom
```

Replace `<ORG>` and `<tenant>-bom` with the values you chose (for example
`git clone git@github.com:mycompany/apollo-bom.git`).

### 3. Fill in your system config

1. Edit **`deploy_targets.yml`**
   - Set `bom:` to the BOM version file you will use (start with `0.1.0`).
   - Optionally set `handlers_bom:` if you use external handlers.
   - Set `helper.repository` / `helper.ref` (defaults point at
     `renglo/bom-helper`; pin a tag once you cut helper releases).
   - Replace the sample `example` tenant with your AWS account id, region,
     and which stages (`staging` / `production`) are enabled.
2. Edit **`bom/v0.1.0.json`**
   - Put real `python`, `npm`, and/or `repos` pins for your system.
   - Do not invent package versions that are not published yet.
3. Optionally edit or remove **`handlers_bom/`** if you do not need it.

### 4. Commit and push

```bash
git add deploy_targets.yml bom/ handlers_bom/
git commit -m "Configure <tenant> BOM"
git push origin main
```

### 5. Before the first real deploy

GitHub Actions in this repo deploy when you push to `main`. For that to work
you still need (outside this README’s scope):

- AWS + launcher/OIDC set up so this GitHub repo is trusted to assume deploy roles
- GitHub Environments / secrets the workflows expect (for example staging)
- Access for Actions to clone **`renglo/bom-helper`** (public today; if it were
  private, grant the org workflow access or use a token)

Until those are ready, you can still push config commits; deploy jobs may fail
until infrastructure is wired.

---

## Files you own

| File | Role |
|------|------|
| `bom/vX.Y.Z.json` | Backend + console package / git pins |
| `handlers_bom/vX.Y.Z.json` | Optional external handlers pins |
| `deploy_targets.yml` | Which versions to deploy, registry, tenants, helper pin |
| `.github/workflows/*` | Deploy workflows (leave as-is unless you know why) |
| `.github/actions/setup-bom-helper` | Checks out bom-helper in CI (leave as-is) |

Do **not** add a `scripts/` folder or a `Dockerfile` to this repo. CI does:

```yaml
- uses: actions/checkout@v4
- uses: ./.github/actions/setup-bom-helper   # checkouts renglo/bom-helper
```

---

## Optional: git-convoy

If your team uses [git-convoy](https://github.com/renglo/git-convoy) to draft
BOM pins from a release train:

```bash
git convoy adopt --bom path/to/<tenant>-bom
# then commit and push inside that *-bom repo → CI deploys
```

You can also edit the JSON / YAML by hand; convoy is optional.

## Optional: local BOM validation

Only needed if you want to preview what CI would install, without deploying.
Clone [bom-helper](https://github.com/renglo/bom-helper) next to your BOM repo
(or anywhere), then:

```bash
python3 /path/to/bom-helper/scripts/bom_manifest.py --plan --pipeline backend bom/v0.1.0.json
python3 /path/to/bom-helper/scripts/bom_manifest.py --validate --pipeline console bom/v0.1.0.json
```

You do **not** need bom-helper checked out locally just to push a `*-bom` repo;
Actions pulls it when workflows run.
