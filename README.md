# amaranta-shop-security

> Transversal security microservice: identity, sign-in and JWT issuing (Annex J)

Part of the **Amaranta Shop** distributed system — team `amaranta-shop`, Grupo 2.
Governance and documentation live in [`amaranta-shop-docs`](https://github.com/code-corhuila/amaranta-shop-docs).

## Branching

Three permanent branches. **None of them accepts a direct commit** — you enter through a child
branch and leave through a Pull Request.

```
develop  <--PR--  feat/... fix/... chore/...
qa       <--PR--  qa/...
main     <--PR--  release/...  hotfix/...
```

Promotion happens **by re-application** (`git cherry-pick -x`), never by merging one permanent
branch into another: `merge develop -> qa` and `merge qa -> main` do not exist in this model.

`main` requires **1 approval from `ariel5253`**. On `develop` and `qa` the team sets its own review
rule.

Full policy: `00-governance/branching-policy.md` in `amaranta-shop-docs`.

## Status

**Design recorded, implementation pending.** The design is [ADR-011](https://github.com/code-corhuila/amaranta-shop-docs/blob/main/05-architecture/decisions/records/ADR-011-identity-design-with-revocation.md);
the service is built from its own spec, which also decides its language (ADR-012). Nothing here
runs yet, and this README says so on purpose.

## What this service owns

The security service is the **only** place that creates accounts, checks credentials, issues
tokens and revokes them (Annex J, J.1.2). It has **no endpoint that signs claims supplied by a
caller**: a token is issued only as the result of a successful login or refresh.

| Piece | Decision (ADR-011) |
|---|---|
| Access token | JWT signed with RS256, 15 minutes, claims `sub`, `role`, `sid`, `jti`, `iat`, `exp` |
| Refresh token | Opaque, at least 256 bits, stored **hashed**, rotated on every use |
| Reuse detection | A rotated refresh token presented again revokes the whole session |
| Revocation | Log out, log out everywhere, and an administrator revoking a user's sessions |
| Window | An access token already issued lives until it expires (at most 15 minutes) |
| Roles | One role per user |

Every other service verifies the token itself with the **public key only** (ADR-009); none of
them can mint a token, and none calls this service on each request.

## Data

PostgreSQL, schema `security`, login user `security_app`, in the single instance of
`amaranta-shop-infra-postgres` (ADR-010). The structure is migrated by `amaranta-shop-security-db`,
never by this repository.

## Configuration

`.env.example` lists every variable, with names and placeholders only. The private key is
mounted as a file and never versioned.
