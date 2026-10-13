# uv sync cannot resolve shared workspace member between two workspace roots (CI log attached)

Issue: xiangmaster/uv#3

Classification: enhancement

## Summary

The reporter says that `uv sync --locked` on Ubuntu 24.04 with uv 0.9.20 cannot resolve a
workspace member that is also the workspace root of another project. The supplied log contains
only the generic error `failed to resolve workspace member \`member-a\``; it does not include the
workspace `pyproject.toml` files, member paths, lockfile, or a verbose uv trace, so the exact
workspace topology and configuration are not established.

The repository source documents and enforces an important boundary: a workspace member with its
own `tool.uv.workspace` table is rejected as a nested workspace. Separately, a project may depend
on a package in another workspace by using a workspace source whose path points to the external
workspace root. The implementation discovers that external root and selects the named member, and
the `lock_external_workspace_source` integration test covers this supported layout.

No matching issue or pull request exists in this checkout's `xiangmaster/uv` repository. The
closest upstream discussions concern workspace discovery boundaries, but none establishes that
the reported shared-root arrangement is supported or that this is a regression.

## Draft response

Thanks for the report. uv does not support using a workspace root as a nested member of another
workspace, so a layout with two workspace roots sharing a member is not currently a supported
workspace configuration. If one project needs a package from another workspace, declare the source
against the other workspace root, for example `pkg = { workspace = "../other-workspace" }`, rather
than making that root a member; the path must identify the workspace root, not the member
directory.

If this is a different layout, please provide the relevant `pyproject.toml` files for both roots
and the member, the complete `uv sync --locked -v` output, and the lockfile (with private URLs or
credentials removed). That will let us determine whether the behavior is an unsupported nested
workspace or a separate resolution bug.

## Classification

This is classified as an enhancement because the requested shared/nested workspace topology is a
new workspace capability, while the current source explicitly rejects nested workspaces. The
report does not provide enough configuration to establish that uv is mishandling a supported
layout, and it does not identify a regression. If the requested layout is intended to use an
external workspace source instead, the source and existing integration test show that supported
form already exists; the missing configuration prevents a more specific bug classification.

## Related

- astral-sh/uv#21305 — open enhancement requesting workspace members outside the workspace root. It is the closest capability discussion because both reports involve workspace layouts spanning separate roots, but its reproduction has one workspace root with a member outside it rather than a member shared by two workspace roots.
- astral-sh/uv#13298 — open bug/needs-decision issue about nested packages and workspace discovery. It covers a single workspace with an intermediate project and explicitly differs from the reported two-root arrangement.
- astral-sh/uv#16640 — open bug about a deeper nested package failing to find its outer workspace root. It is a related discovery-boundary case, not evidence that shared workspace roots are supported.
- astral-sh/uv#19916 — closed bug for a 0.11.20 regression involving an intermediate `pyproject.toml` and workspace source overrides. Its behavior and version range differ from this report, so it is not the same issue.
- astral-sh/uv#19926 — merged fix for astral-sh/uv#19916. It restores the intermediate-`pyproject.toml` behavior and does not add support for shared or nested workspace roots.

## Supporting evidence

- `crates/uv-workspace/src/workspace.rs` defines `NestedWorkspace` and returns that error when a
  discovered non-root member has its own `tool.uv.workspace` table.
- `crates/uv-distribution/src/metadata/lowering.rs` handles `workspace = "..."` by discovering
  the target workspace, requiring the path to be that workspace's root, and selecting the named
  member.
- `crates/uv/tests/lock/lock.rs` contains `lock_external_workspace_source`, which successfully
  locks a project depending on a member of an external workspace and separately snapshots the
  error for a path aimed at the member instead of the root.
- The `xiangmaster/uv` issue search returned only open issues #1, #2, and #3 and no pull requests;
  #1 concerns proxy download concurrency and #2 concerns git workspace lock hashes, so both were
  ruled out as unrelated.
