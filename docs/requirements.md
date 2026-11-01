# Toolchain lab: requirements

A mock GitHub org to test central control of CI/CD without DevOps becoming a
bottleneck. Starred items are in scope for the first build.

## Mandatory, cannot be skipped

1. * Secret scan and SonarQube run on every PR to `main`. Teams cannot remove or skip them from their own `ci.yml`.
2. An image must pass its vulnerability scan and come from the trusted build before it deploys.
3. * Prod deploys only through the central deploy template.
4. * Prod deploys are approved by the owning team, not the author.
5. Third-party actions come from an allow-list and are pinned to a commit SHA.

## Custom, added safely

6. * A team adds its own jobs (e2e, lint, nightly) without asking DevOps.
7. * A team configures a template through inputs, without forking it.
8. * A team adds custom steps to a standard build without being able to tamper with mandatory checks.

## Shapes and tiers

9. * Vendor app: no source, only package, scan and deploy an image. Mandatory gates still apply.
10. Tiers: critical repos get a stricter mandatory set than internal tools.

## Exceptions

11. * Break-glass: a time-limited, logged bypass for a hotfix, approved outside the team.
12. Waivers: an accepted CVE with an expiry date.

## The platform itself

13. * Non-breaking template fixes reach every repo with no PRs. Breaking changes are opt-in with a deprecation window.
14. App teams cannot change templates or move their tags.

## Visibility and onboarding

15. Central view of compliance, template versions and bypasses used.
16. Audit trail per prod deploy: approver, commit, scans passed.
17. A new repo is compliant from its first commit, with no tickets.

## Repo manifest

18. * A central manifest lists every approved repo with its owning team, contacts, tier and shape.
19. * Repos are created only from a manifest PR. The owning team approves. Platform approval is off by default and can be turned on per team.
20. Prod deploys are refused for repos not active in the manifest.

## Enforcement

21. A mandatory check counts only if the team cannot edit what produces it.
