# Qryptive — PQC Vulnerability Scanner (GitHub Action)

Detect quantum-vulnerable cryptography (RSA, ECDSA, ECDH, DES, SHA-1, MD5, weak TLS, and more) in your Python, Java, Go, Terraform, CloudFormation, and Kubernetes manifests.

**The scanner runs inside your GitHub Actions runner.** In the default mode (no API key), no source code or scan data is sent to Qryptive — findings appear only as GitHub PR annotations and SARIF in your Security tab. This is the privacy posture banks and other regulated industries require.

Full data-handling document: [`docs/ENTERPRISE_PRIVACY.md`](https://github.com/qryptive/.github/blob/main/docs/ENTERPRISE_PRIVACY.md).

---

## Quick start — local-only mode (no account required)

```yaml
# .github/workflows/pqc-scan.yml
name: PQC Scan
on: [push, pull_request]
permissions:
  pull-requests: write
  security-events: write
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: qryptive/pqc-scanner-action@v1
        with:
          fail_on_vulnerable: true
          severity_threshold: high
          report_format: sarif
      - uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: pqc-results.sarif
```

Findings appear as:
- Inline annotations on PR diffs
- Code scanning alerts in the GitHub **Security tab**
- Markdown summary table in the Actions run log

---

## With AI-enhanced disambiguation (opt-in, paid)

Reduces false positives via LLM-based analysis of ambiguous findings. Finding metadata is sent to Qryptive (file paths, line numbers, algorithm names, ~10-30 lines of context per finding). The repository as a whole is never uploaded.

```yaml
- uses: qryptive/pqc-scanner-action@v1
  with:
    pqc_api_key: ${{ secrets.PQC_API_KEY }}
    fail_on_vulnerable: true
```

Sign up at https://qryptive.ai to get an API key.

---

## With automated fix PRs (paid)

When vulnerabilities are found, a PR is automatically opened with the post-quantum migration applied.

```yaml
- uses: qryptive/pqc-scanner-action@v1
  with:
    pqc_api_key: ${{ secrets.PQC_API_KEY }}
    github_token: ${{ secrets.GITHUB_TOKEN }}
    migration_mode: pqc_only  # or staged_rollout, hybrid
```

---

## Inputs

| Input | Default | Description |
|---|---|---|
| `paths` | `**/*.py **/*.java **/*.go` | Glob patterns to scan |
| `scan_scope` | `changed` | `changed` (PR/push diff) or `all` (entire repo) |
| `severity_threshold` | `medium` | `critical`, `high`, `medium`, `low` |
| `fail_on_vulnerable` | `true` | Fail the build if vulnerabilities found |
| `report_format` | `annotations` | `annotations`, `sarif`, or `json` |
| `pqc_api_key` | — | Opt-in to AI disambiguation + dashboard |
| `migration_mode` | `pqc_only` | `pqc_only`, `staged_rollout`, `pqc_preferred`, `hybrid` |
| `github_token` | — | Required for fix PR creation (`secrets.GITHUB_TOKEN`) |

---

## Languages and frameworks

| Language | AST-based | Patterns detected |
|---|---|---|
| **Python** | Yes (LibCST) | hashlib, cryptography, pycryptodome, ssl, hmac, JWT, dataflow |
| **Java** | Yes (JavaParser) | JCA, BouncyCastle, JWT (jose4j/nimbus/jsonwebtoken), Spring Security |
| **Go** | Yes (go/ast) | crypto stdlib, x509, jwt-go, gosec patterns |
| **IaC** | Pattern | Terraform, CloudFormation, Kubernetes, Helm — KMS keys, TLS policies, certificate references |

---

## Example repository

See [`qryptive/example-vulnerable-app`](https://github.com/qryptive/example-vulnerable-app) for a complete working setup with intentionally vulnerable code samples and the workflow file above.

---

## Privacy & security

| Mode | What leaves your runner |
|---|---|
| Local-only (no `pqc_api_key`) | **Nothing.** Zero outbound calls to Qryptive. |
| With `pqc_api_key` (opt-in AI) | Finding metadata + ~10-30 lines of code snippet around each finding. Repository as a whole is never uploaded. |

To prove zero egress in local-only mode, add an egress network policy to your runner that blocks all destinations except `*.github.com` — the action will succeed.

---

## License

The action.yml metadata in this repo is MIT-licensed. The compiled scanner image is proprietary — see https://qryptive.ai/terms.
