# Hugging Face Spaces Keepalive

This workflow periodically pings configured Hugging Face Spaces with lightweight HTTP requests. Its goal is to help reduce inactivity-based sleeping on Spaces that may be idle between real usage. It does not authenticate to Hugging Face, and it does not guarantee permanent uptime.

## What It Does

- Checks multiple Hugging Face Spaces on a fixed schedule.
- Records HTTP status, approximate response time, and per-Space success or failure.
- Continues checking the remaining Spaces if one Space fails.
- Fails the GitHub Actions run when at least one Space is unreachable or returns an error status.
- Uses only built-in GitHub Actions capabilities, Bash, and `curl`.

## Project Structure

- `.github/workflows/huggingface-keepalive.yml`
- `README.md`

## Configured Spaces

The workflow is configured with these example Spaces:

- `ava2lon-litellm|https://ava2lon-litellm.hf.space/`
- `ava2lon-flux|https://ava2lon-flux.hf.space/`

## How To Add Another Space

Edit `.github/workflows/huggingface-keepalive.yml` and add another `name|URL` entry inside the embedded `SPACES` block.

Example:

```bash
SPACES=(
  "ava2lon-litellm|https://ava2lon-litellm.hf.space/"
  "ava2lon-flux|https://ava2lon-flux.hf.space/"
  "video|https://your-video-space.hf.space/"
)
```

Use any display name you want. The workflow parses the name and URL from each line, so adding or removing entries does not require changes elsewhere in the script.

## How To Manually Run

In GitHub:

1. Open the repository's **Actions** tab.
2. Select **Hugging Face Spaces Keepalive**.
3. Click **Run workflow**.
4. Choose the branch that contains the workflow.
5. Click **Run workflow**.

This uses `workflow_dispatch`, so no secrets or approval flow is required for an outbound HTTP health check.

## Schedule

The workflow runs every 30 minutes using:

- cron: `*/30 * * * *`

A scheduled run can be skipped if the previous run is still in progress because the workflow uses non-canceling concurrency.

## Failure Reporting

If a Space is unreachable or returns an HTTP error, the workflow logs output like this:

- Checking: `<space-name>`
- URL: `<space-url>`
- HTTP Status: `<status-code-or-000>`
- Response Time: `<time>`
- Result: `FAILED`

The final summary block also lists failed Spaces. The job exits non-zero when any Space fails.

## Why No Hugging Face Token Is Required

The keepalive request is a public HTTP check. It does not:

- require a `HF_TOKEN`
- require a `HUGGINGFACE_TOKEN`
- use `LITELLM_MASTER_KEY`
- use `LITELLM_API_KEY`
- use `OPENAI_API_KEY`
- or use any other secret

Do not add secrets merely for HTTP health checks. The workflow is intentionally read-only.

## Known Limitations

- Keepalive requests do not guarantee permanent uptime.
- Hugging Face controls Space lifecycle behavior, cold starts, and hardware behavior.
- A successful keepalive request only shows that the Space responded during that check.
- If a Space is down for longer than the schedule interval, the next successful run does not prove continuous uptime.
