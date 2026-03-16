# Post Benchmark Report Action

A reusable GitHub Action that uploads a benchmark JSON report to a backend HTTP service via multipart form POST.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `backend_url` | **yes** | — | Full URL of the backend metrics endpoint |
| `report_path` | **yes** | — | Path to the benchmark JSON report file |
| `api_token` | no | `""` | Bearer token for authentication |
| `file_format` | no | `json` | File format identifier |
| `file_type` | no | `benchmark` | File type identifier |
| `is_zipped` | no | `false` | Whether the file is zipped |
| `git_project_name` | no | `${{ github.repository }}` | Project name |
| `workflow_name` | no | `${{ github.workflow }}` | Workflow name |
| `job_name` | no | `${{ github.job }}` | Job name |
| `pr_id` | no | auto-detected | PR number (auto-detected from PR events, defaults to `0` for push events) |
| `header_config` | **yes** | — | JSON array of header config items (see below) |
| `list_code` | **yes** | — | List code identifier |
| `list_name` | **yes** | — | List display name |
| `fail_on_error` | no | `true` | Whether to fail the step on upload error |

## Outputs

| Output | Description |
|---|---|
| `status` | HTTP status code from the upload request |

## Usage

### Basic (in a benchmark workflow)

```yaml
steps:
  - name: Run benchmark
    id: benchmark
    run: |
      python run_benchmark.py --output benchmark_metrics.json

  - name: Upload benchmark report
    if: steps.benchmark.outcome == 'success'
    uses: flagos-ai/FlagOps/actions/post-benchmark-report@main
    with:
      backend_url: 'http://10.1.4.167:30180/flagcicd-backend/metrics/'
      report_path: 'benchmark_metrics.json'
      list_code: 'benchmark-list'
      list_name: 'Benchmark Results'
      header_config: |
        [{"field":"metric","name":"Metric","required":true,"sortable":true,"type":"string"},
         {"field":"value","name":"Value","required":true,"sortable":true,"type":"number"}]
```

`workflow_name`, `job_name`, `pr_id`, and `git_project_name` are auto-detected from the GitHub context.

### With authentication and custom settings

```yaml
  - uses: flagos-ai/FlagOps/actions/post-benchmark-report@main
    with:
      backend_url: 'http://10.1.4.167:30180/flagcicd-backend/metrics/'
      report_path: 'benchmark_metrics.json'
      is_zipped: 'true'
      api_token: ${{ secrets.BACKEND_TOKEN }}
      job_name: 'benchmark_tests'
      fail_on_error: 'false'
```

### Non-PR context (push to main)

When running outside a PR context, `pr_id` defaults to `0`:

```yaml
  - uses: flagos-ai/FlagOps/actions/post-benchmark-report@main
    with:
      backend_url: 'http://10.1.4.167:30180/flagcicd-backend/metrics/'
      report_path: 'benchmark_metrics.json'
      pr_id: '0'  # optional explicit override
```

## `header_config` Format

`header_config` is a JSON array describing the columns of the report list. Each item has the following fields:

| Field | Type | Description |
|---|---|---|
| `field` | string | Column field key |
| `name` | string | Column display name |
| `required` | boolean | Whether the column is required |
| `sortable` | boolean | Whether the column is sortable |
| `type` | string | Data type (`string`, `number`, etc.) |

Example:

```json
[
  {
    "field": "username",
    "name": "用户名",
    "required": true,
    "sortable": true,
    "type": "string"
  }
]
```

## Behavior

- **Auto-detection**: `git_project_name`, `workflow_name`, `job_name`, and `pr_id` are auto-populated from GitHub context variables. Any of them can be overridden by setting the input explicitly.
- **pr_id fallback**: If no PR context is available and `pr_id` is not set, it defaults to `0`.
- **Error handling**: Controlled by `fail_on_error`. When `true` (default), a failed upload or missing report file fails the workflow step. When `false`, a warning is logged and the step succeeds.
