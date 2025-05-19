# 🚀 Cloudflare Pages Deployment via GitHub Tags

This setup ensures:

* Every **versioned tag (`v*`)**, **hotfix (`hotfix-*`)**, or **rollback (`rollback-*`)** deployment is deployed to **production**.
* Rollbacks can be done easily using `git checkout` and re-tagging with a `rollback-*` tag.
* Static files in `./public` are deployed directly using `wrangler`.

---

## 🔧 Prerequisites

### 1. Install and Configure `wrangler.toml`

> Root of your repository

```toml
# wrangler.toml
name = "github-action-tag"
pages_build_output_dir = "./public"
```

### 2. Create a Cloudflare API Token

Generate token here:
🔗 [Cloudflare API Token Settings](https://dash.cloudflare.com/profile/api-tokens)

**Required permissions:**

* **Account → Cloudflare Pages → Edit**
* **Zone → Read**
* (Optional) Restrict by account or project

Store it in your GitHub repo:

```
Settings > Secrets and variables > Actions > New repository secret
Name: CLOUDFLARE_API_TOKEN
```

---

## 🛠️ GitHub Actions Workflow

> `.github/workflows/cf.yaml`


```yaml
name: Deploy Tagged Release to Cloudflare Pages

on:
  push:
    tags:
      - 'v*'
      - 'rollback-*'
      - 'hotfix-*'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Tag
        uses: actions/checkout@v3
        with:
          ref: ${{ github.ref }}

      - name: Set Up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install Wrangler
        run: npm install -g wrangler

      - name: Deploy to Cloudflare Pages (Production)
        env:
          CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
        run: |
          wrangler pages deploy ./public \
            --project-name github-action-tag \
        #verify this in pages is set to production
            --branch main 

```

---

## 🚀 Deployment Usage

### 🔄 Normal Deployment

```bash
# Make changes
git add .
git commit -m "feature added"
git push origin main

# Tag release
git tag v11
git push --tags
```

> ✅ Automatically deploys tag `v11` to production (Cloudflare Pages → `main` branch)

---

## 🧯 Rollback to Previous Tag

### Step-by-step:

```bash
# Suppose v11 caused issues, and v10 was stable

git checkout v10          # Checkout the working tag
git tag rollback-v10      # Create rollback tag
git push --tags           # Push rollback tag
```

> ✅ This will **re-trigger the deployment** to production using the `v10` commit

---

## 🧪 Example Tags You Can Use

| Tag Name      | Purpose             |
| ------------- | ------------------- |
| `v1`, `v2`    | Normal release      |
| `hotfix-v2`   | Emergency patch     |
| `rollback-v1` | Rollback to old tag |

---

## 📝 Notes

* Cloudflare uses the `--branch main` flag to treat this as a **production deployment**.
* Tags point to commits, so you can roll back by creating a new tag from an older commit.
* Wrangler uses `./public` as the deployment directory (you can customize via `wrangler.toml`).
* You must use **Node.js v20+** as required by Wrangler 4.x.

---

## ✅ Final Checklist

* [x] `wrangler.toml` with `pages_build_output_dir`
* [x] GitHub secret: `CLOUDFLARE_API_TOKEN`
* [x] GitHub Action workflow deployed on tag
* [x] Use `rollback-*` tags to trigger safe rollback

