# CP2 - Docker build evidence

Date: 2026-09-29 (Asia/Saigon)
Docker: Docker version 29.8.0, build 88096ef

## Test suite

Command: `python -X utf8 -m pytest tests/test_cp2.py -q`
Result: `16 passed in 3.66s` (recorded during the CP2 checkpoint run).

## Image comparison

The one-stage baseline was restored from the repository's original Dockerfile at commit `1bf8ea5` (`FROM python:3.11`, root user, `COPY . .`, then `pip install -r requirements.txt`) and built as `day12-agent:single-original`.

| Build | Docker tag | `docker images` | Exact bytes |
|---|---|---:|---:|
| Original one-stage | `day12-agent:single-original` | 1.73 GB | 1,731,661,290 |
| Current multi-stage | `day12-agent:cp2-test` | 275 MB | 275,005,204 |

The multi-stage image saves 1,456.7 MB (about 84.1%). It uses `python:3.11-slim` stages and does not carry the full one-stage base or the pip download cache into runtime.

## Reproduce

The baseline source is available with `git show 1bf8ea5:Dockerfile`. It was
written as UTF-8 without a BOM to a temporary file, then built with
`docker build -f Dockerfile.single.tmp -t day12-agent:single-original .`.
The production image was built from the current `Dockerfile` by the CP2 test.

The temporary baseline Dockerfile was removed after measurement; the original remains in Git history at `1bf8ea5:Dockerfile`.
