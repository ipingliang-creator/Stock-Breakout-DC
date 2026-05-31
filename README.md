# Stock-Breakout-DC — public data repo

This folder holds the files to drop into the **public** data-collection repo
(`Stock-Breakout-DC`). The split (see [`../../docs/DEPLOYMENT.md`](../../docs/DEPLOYMENT.md)):

- **Public repo (Stock-Breakout-DC):** runs the heavy `update-data` on *unlimited*
  free Actions minutes and commits the cached prices + fundamentals. **Data only —
  no strategy code.**
- **Private repo (this one):** the strategies/scanner/backtester. Consumes the
  public data (read-only) to scan and report.

## One-time setup

1. Create the public repo **`Stock-Breakout-DC`** (public, empty).
2. Add this file as `.github/workflows/data-collect.yml`:
   - copy [`data-collect.yml`](data-collect.yml) there.
3. On `Stock-Breakout-DC` → Settings → Secrets and variables → Actions, add:
   - `PRIVATE_REPO_TOKEN` — a fine-grained PAT with **read** access to
     `ipingliang-creator/stock-breakout-bot` (Contents: Read).
   - `ALPACA_API_KEY`, `ALPACA_SECRET_KEY`, `ALPACA_BASE_URL`
   - `SBS_SEC_USER_AGENT` — your contact email (SEC EDGAR requires it).
4. Run the **Data Collection** workflow (Actions → Run workflow → `provider: alpaca`).
   It commits `data/cache/**` (prices + EDGAR fundamentals) into the repo.

## How the private repo consumes it

The private scan pulls the published cache, then scans offline (no re-fetch). See
`docs/DEPLOYMENT.md` for the exact private-side workflow snippet — in short:

```bash
git clone --depth 1 https://github.com/ipingliang-creator/Stock-Breakout-DC data_pub
cp -r data_pub/data/cache ./data/cache
python -m sbs.cli --provider alpaca --db data/sbs_alpaca.sqlite scan --as-of ""   # reads cache, no fetch
```

> Why a token to read private code? The public repo needs the *data layer* code
> to do the fetching, but we never commit that code here — only `data/`. The
> engine is checked out into `_engine/` transiently per run.
