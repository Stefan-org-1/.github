# Toolchain lab: design

## Policy gates

CODEOWNERS can only say "any one of these approves". Rules like "a sentinel of the owning team and a manager" need a custom gate.

The gate is a platform GitHub App. Only the holder of its private key can act as it, and GitHub records its App ID on everything it posts. That is the mark nobody else can forge.

| Gate | Mark | Enforced by |
|---|---|---|
| Merge to `main` | Check run `policy`, posted by the App | Repo ruleset requiring `policy` from the App's ID |
| Deploy to `prod` | Approval of the deploy job, given by the App | Custom deployment protection rule on the `prod` environment |

People approve merges with normal PR reviews. CODEOWNERS routes review requests. Deploys are approved in a change request instead. The App decides whether the right combination approved, and checks the repo is active in the manifest.

Pipelines do not check the mark. GitHub enforces it before the merge button works and before the deploy job gets secrets.

### Threats

| Threat | Defence |
|---|---|
| Same-named check from a team workflow | Ruleset pinned to the App's ID. Actions posts as a different App. |
| Team edits its workflow to skip the gate | Gates live in the ruleset and environment, outside the workflow |
| App private key leaks | Only platform holds it. Rotate. App runs outside team repos. |
| Org admin bypasses rulesets | Bypass list limited to break-glass, audit log review |
| Image built elsewhere is deployed | Signed build provenance. Cloud trusts only the central deploy workflow through OIDC. |

### Runtime

1. Mock: a scheduled workflow in `platform-gate` signs in as the App, polls open PRs and waiting deploys, posts results. Minutes of delay. Verify the deployment protection rule API accepts a polled answer.
2. Real: the App receives webhooks on a serverless endpoint. Same rule logic.

If the App is down, checks stay pending and deploys wait. The gate fails closed.

## Approvers

- `team-<area>`: product teams. Own their code.
- `group-sentinels`, `group-managers`: cross-team groups.
- `sentinels-<area>`, `managers-<area>`: nested under the groups. Used where approval must come from the owning team, such as `prod` reviewers.
- Membership is declared in `org-manifest` and applied by `org-terraform`.

## Trusting results from team pipelines

A mandatory check that runs inside a team's pipeline counts only if:

1. **Isolation:** it runs in its own job, on a fresh runner, from a fresh checkout of the exact commit, with no team code and no caches or artifacts a team job wrote.
2. **Attestation:** it signs its result. Sigstore puts the identity of the workflow file that ran the job into the signing certificate. The gate verifies the signer is the central template.

Job names and job conclusions are not enough: a team can add a job with the same name.

### Tested on payments-web#2 (node-build v2)

| Attempt | Result |
|---|---|
| Pre-build hook puts a fake `git` on `PATH` | Worked in the `build` job, including on the checkout action's own cleanup. No effect on `scan`. |
| Pre-build hook overwrites `SHA` through `$GITHUB_ENV` | Same: `build` only. |
| Team `forge` job signs a "scan passed" result with the same predicate type | Signed as `payments-web/.github/workflows/ci.yml`. Rejected by `--signer-workflow`. |
| Template `scan` job result | Verified, signed as `ci-templates/.github/workflows/node-build.yml@refs/tags/v2`. |

### Consequences

- Hooks go in jobs that hold no mandatory checks. A step in the same job as a check can always tamper with it.
- Requiring `id-token: write` from callers made v2 a breaking change.
- Public repos need no App permission to read attestations. Private repos need Attestations: read.
- The subject is a file holding `<repo>@<sha>`, because attestations sign file or image digests, not commits.
