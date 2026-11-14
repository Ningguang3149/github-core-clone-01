# github-core-clone-01 — Single-Process Git Forge (`ningguang3149`)

A self-contained GitHub-compatible forge written in TypeScript (`app.ts`) relying solely on `node:http`, `node:crypto`, and `node:zlib`.

## Key Design & Speed Enhancements

- **Packfile Stream Consumption (`bytesWritten`) & Object-Level Deflate Cache**:
  - `readPack` inflates each pack entry in a single pass via `inflateSync(..., { info: true })` and advances by `engine.bytesWritten`.
  - `writePack` stores the pre-compressed `packEntry` on each `GitObject` so subsequent `git-upload-pack` fetches reuse deflated object chunks.
- **Tree Closure & Flat-Tree Memoization**:
  - `treeClosureOids` caches the reachable object set (`Set<string>`) per `treeOid` in `repo.treeClosureCache`, eliminating repeated subtree walks during `selectFetch` negotiation.
  - `flattenTree` caches flattened file maps per `treeOid` in `repo.flatTreeCache` for fast three-way diff and conflict detection.
- **GraphQL v4 AST Document Cache**:
  - `parseCachedGraphQl` caches parsed `GqlOperation[]` ASTs by query string in `gqlDocCache`, avoiding repeated tokenization and parsing across parameterized and batched GraphQL requests.
- **Supported Surfaces**:
  - **Git Smart HTTP**: `/{owner}/{repo}.git/info/refs`, `git-upload-pack` (protocol v2 `ls-refs`, `fetch`, `shallow`, `deepen`), and `git-receive-pack` (v0 `report-status`, `side-band-64k`, `delete-refs`, fast-forward verification).
  - **REST v3**: `/api/v3/...` endpoints for repositories, branches, commits, refs, file contents, labels, issues, comments, pull requests, and merges.
  - **GraphQL v4**: `POST /api/graphql` with `viewer`, `repository`, `node`, `__type` introspection, cursor connections, and mutations (`createIssue`, `closeIssue`, `createPullRequest`, `mergePullRequest`, `addComment`).
  - **MCP**: `POST /mcp` JSON-RPC 2.0 streamable HTTP (`2026-07-28`) providing `create_repo`, `get_repo`, `create_issue`, `list_issues`, `create_pull_request`, and `merge_pull_request`.
