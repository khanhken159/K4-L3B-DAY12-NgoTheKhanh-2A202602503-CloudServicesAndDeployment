# GitHub Actions bonus evidence

- Workflow: `.github/workflows/ci.yml`.
- Triggers: push to `main` and pull requests targeting `main`.
- `test` installs `requirements.txt` and runs CP1–CP4 plus workflow checks.
  CP5 is omitted because it calls the live Railway service; the badge status
  check is omitted from the workflow run itself to avoid self-reference.
- `build` builds the production Dockerfile on GitHub's Ubuntu runner.
- `deploy` needs both `test` and `build`, and is restricted to a push to `main`.
  It runs only after `RAILWAY_DEPLOY_ENABLED=true`; the Railway project token is
  read from the `RAILWAY_TOKEN` repository secret.
- Local structural verification:
  `python -m pytest tests/test_bonus_cicd.py -k "not badge_bao_passing" -q`
  → `12 passed, 1 deselected`.
- Combined local verification:
  `python -m pytest tests/test_cp1.py tests/test_cp2.py tests/test_cp3.py tests/test_cp4.py tests/test_cp5.py tests/test_bonus_cicd.py -q -rs -k "not badge_bao_passing"`
  → `91 passed, 4 skipped, 1 deselected`. The four skips are LOCAL_FALLBACK-only
  tests; the deselection is the live badge check, which requires a completed
  GitHub Actions run.

The live passing badge can only be verified after the workflow is pushed to the
public GitHub repository and its run completes. Do not treat the deselected
badge check above as a passing remote workflow run.
