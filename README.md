# uv lock hash differs across machines with git workspace dep (with maintainer triage directive)

Issue: xiangmaster/uv#2

Classification: bug

## Summary

The reporter says that `uv lock` produces different hashes across machines when the project uses a
`git+https` workspace dependency, even when `--locked` is supplied. The issue body does not include
the referenced lockfiles, a public tracker link, a reproduction repository, uv versions, platform
details, or a lockfile diff.

The repository's source and issue history show that Git-backed workspaces have had bugs involving
source identity and cache-checkout paths. Those historical reports were fixed, but none establishes
that this report is the same problem or that the behavior has regressed. A moving branch or tag,
different Git state, or environment/configuration differences are also possible explanations that
cannot be evaluated without the missing artifacts.

## Draft response

Thanks for reporting this. Different `uv.lock` hashes for the same project and git+https workspace
dependency would be incorrect, but the issue currently does not include the referenced lockfiles or
enough information to determine the cause. Please provide a minimal public reproduction (or the
relevant `pyproject.toml` and lockfile sections), the exact `uv lock` command, `uv --version` and
platform for each machine, and a diff of the generated lockfiles. Please also say whether the Git
dependency uses a branch, tag, or fixed revision. That will let us distinguish a source-
identity/checkout-path difference from a moving Git reference or environment/configuration
difference.

## Classification

This is classified as a bug because the reported behavior is a correctness and reproducibility
failure: the same project and Git workspace dependency allegedly produce different lock hashes
across machines under `--locked`. It is not a request for new functionality. The report does not
provide enough evidence to confirm the mechanism or identify a regression, but a complete
reproduction is not required to classify an asserted incorrect behavior as a bug.

## Related

- astral-sh/uv#8887 — Reports a Git-backed workspace that worked locally but failed when added from Git. Maintainers identified that workspace members discovered inside a Git checkout could be represented as cache-local directory URLs, producing conflicting or invalid lockfiles. This is the closest Git-workspace source-identity discussion, but its symptom is different.
- astral-sh/uv#9388 — Merged fix for astral-sh/uv#8887 that preserves Git member information when metadata is read from cache. It addresses Git-versus-directory source identity, not a confirmed cross-machine hash difference.
- astral-sh/uv#19152 — Reports a machine-specific absolute path in `uv.lock` for a transitive Poetry relative path dependency inside a Git checkout. It has the same portability concern, but a different dependency format and trigger.
- astral-sh/uv#19269 — Merged fix for astral-sh/uv#19152 that preserves Git source identity and serializes a Git subdirectory instead of an absolute checkout path. It is a relevant historical fix, not an established fix for this report.
- astral-sh/uv#14954 — Open issue about Git commit hashes for multiple subdirectory dependencies from one repository not being updated selectively. It shares the Git subdirectory and commit-hash area, but concerns upgrade semantics rather than machine-to-machine divergence under `--locked`.
- astral-sh/uv#13707 — Cross-machine lockfile drift caused by registry index URL trailing-slash differences. It is a close reproducibility symptom, but does not involve Git workspace sources.
- astral-sh/uv#14245 — Merged registry URL normalization fix for astral-sh/uv#13707; it does not establish a fix for Git workspace hash divergence.

## Supporting evidence

- The new issue is open at xiangmaster/uv#2 and has no comments or labels.
- xiangmaster/uv#1 was inspected as the only other issue in the fork; it concerns `uv sync` download concurrency behind an NTLM proxy and is unrelated.
- astral-sh/uv#8887 and astral-sh/uv#9388 establish a historical Git-workspace metadata/cache identity problem and its fix.
- astral-sh/uv#19152 and astral-sh/uv#19269 establish a historical machine-specific Git-checkout path problem and its fix.
- astral-sh/uv#13707 and astral-sh/uv#14245 establish a separate registry URL normalization problem and fix.
- No existing issue or pull request inspected tracks the exact combination of differing machine-generated hashes, a `git+https` workspace dependency, and `--locked`.

## Search scope

The fork's open and closed issues and open and closed pull requests were searched separately using
literal terms such as `hash differs`, `different hashes`, `across machines`, `git+https`,
`--locked`, `git workspace`, and `workspace dependency`. Conceptual searches covered lockfile
reproducibility, Git/VCS revisions, Git subdirectories, source identity, cache paths, and
cross-platform lockfile drift. Fix-oriented searches followed the Git workspace metadata,
transitive Git path, and URL-normalization discussions into their referenced pull requests.

The upstream history was also inspected because the checkout is a fork. The especially plausible
results ruled out were astral-sh/uv#10988 (mixed uv versions), astral-sh/uv#11312 (an
environment-provided registry URL and trailing slash), astral-sh/uv#4317 and astral-sh/uv#11090
(expected retention of a locked Git revision until upgrade), astral-sh/uv#9301 (a setuptools build
failure), and astral-sh/uv#14954 (selective Git dependency upgrade behavior).
