# FestPass — Docker & Container Security Assessment

## Student and release details

| Item | Value |
| --- | --- |
| Name | Akshat |
| REGNO | `202401100200030` |
| PORT | `8030` |
| Docker Hub username | `akkichauhan15` |
| Docker Hub repository | `https://hub.docker.com/r/akkichauhan15/fest-backend` |
| `1.0.0` digest | `sha256:31b1122212aac732271bdab0893baae5b61517dea080d0ecc7d095a51553f054` |

## Architecture

FestPass has an nginx frontend, a Flask backend, and PostgreSQL 17. The frontend and
backend share the `web` network. The backend and database share the internal `data`
network. The frontend is therefore unable to resolve or connect to `db`. PostgreSQL data
is stored in the named `postgres-data` volume.

## Run the stack

The verified images and running stack are in Docker's `default` context. Select it first
with `docker context use default`.

1. The ignored `.env` is already personalized for local assessment use.
2. For Stage 2, build and run locally:

   ```sh
   docker compose -f compose.yaml -f compose.build.yaml up -d --build
   ```

3. After publishing `1.0.0`, run the final registry-backed configuration:

   ```sh
   docker compose up -d
   ```

4. Open `http://localhost:<PORT>`.

## Vulnerability results

These values must be copied from the reports produced on this machine; they are not
invented.

| Image | CRITICAL | HIGH | Image size |
| --- | ---: | ---: | ---: |
| `fest-backend:0.1` | 234 | 2,249 | 1.60 GB |
| `fest-backend:0.2` | 2 | 64 | 210.84 MB |

Reports are stored as `reports/trivy-before.txt` and `reports/trivy-after.txt`.
Counts above combine the OS and Python-package result sections. The two remaining
CRITICAL findings in `0.2` have no available fix; the required command with
`--severity CRITICAL --ignore-unfixed --exit-code 1` exits with code `0`.

## Local verification

- Trivy `0.76.1` scanned both images and the required fixed-critical gate passed.
- Cosign `3.1.4` is installed locally for Stage 5.
- The fixed container runs as UID/GID `10001:10001`.
- Docker Hub tags `1.0.0` and `1.0` resolve to the recorded digest.
- Compose successfully pulled `1.0.0` from Docker Hub after its local tags were removed.
- The health endpoint returned `database: true` through nginx.
- `ping -c 1 db` from the frontend failed with `bad address`, proving network isolation.
- Two local test registrations survived a Compose teardown and restart.

## AI declaration

OpenAI Codex was used to transcribe the supplied starter files, create the Docker and
Compose configuration, explain security controls, and help run local verification. All
commands, screenshots, registry pushes, and signing evidence are produced on my own
machine and accounts.

## Incomplete steps

- Screenshots: must be captured unedited with the personalized REGNO terminal prompt.
- Cosign public key and signature evidence: pending local password-protected key generation.

## Evidence command checklist

Set the required prompt in every evidence terminal first:

```sh
export PS1='[202401100200030] %~ %# '
export PATH="$PWD/.tools:$PATH"
```

Run the commands in `EVIDENCE.md` in order and save the full-window screenshots under
`screenshots/` using the exact evidence code names.
