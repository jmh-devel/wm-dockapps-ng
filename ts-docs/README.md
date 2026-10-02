# Concourse CI learning trail

This directory records the first CI experiment for `jmh-devel/wm-dockapps-ng`.
The working branch is `dev/concourse-ci-lab`, based on `master` commit
`251e851a9d59d8bd0263ba1eebe08196f98f6d0b` (2026-04-12). This is a
public collection of independent, mostly Autotools based C dockapps, not one
top-level build. The first CI exercise should build one representative app,
`wmcalc`, before expanding coverage.

## Current status (2026-10-02)

1. Inspected the repository and the existing `ts-concourse-demo` lab pipeline.
2. Created `dev/concourse-ci-lab` in the local clone. No pipeline has been
   applied and no Concourse build has run for this repository.
3. Opened [tsctl issue #1131](https://github.com/tacitness/tsctl/issues/1131)
   for catalog enrollment. `tsctl repos check wm-dockapps-ng` currently reports
   an unknown repo key, so a governed implementation Job cannot start yet.
4. Fly access was restored on 2026-10-02. The existing
   `ts-concourse-demo-main/verify #2` build succeeded on `learning-lab` for
   demo commit `f1e02d1`; this verifies the lab path, not this repository.
5. GitHub Issues are disabled on this repository. The implementation is
   tracked in [ts-concourse-demo issue #3](https://github.com/jmh-devel/ts-concourse-demo/issues/3).
6. An explicit `codex2` issue-dispatch dry run summarized `codex2` but rendered
   a `codex` Job. [tsctl #1123](https://github.com/tacitness/tsctl/issues/1123)
   tracks this dry-run defect. A live profile mismatch has not been proven;
   dispatch is held until profile selection can be verified. The managed issue
   queue client also needs an approved server URL and machine API capability;
   neither is configured in the current shell.
7. The documentation was published in
   [draft PR #1](https://github.com/jmh-devel/wm-dockapps-ng/pull/1).
8. A temporary Concourse source probe validated the public HTTPS Git resource
   and fetched branch commit `ff7e4b7286cb307161ccc47a77f4adea63d81bff`
   on the `learning-lab` worker. Its
   [build #1](http://127.0.0.1:8080/teams/main/pipelines/wm-dockapps-ng-source-probe/jobs/verify-source/builds/1)
   succeeded. This is source access evidence only; no `wmcalc` build ran.

These are observed states, not successful CI evidence. Update this list with
the PR, exact commit, Fly validation result, and Concourse build URL as the
work progresses.

## First experiment

The proposed `verify-wmcalc` job should fetch a selected Git branch using the
built-in Concourse Git resource and this public repository's HTTPS URL. The
private `ts-concourse-demo` needs a GitHub App; this public repository does not
need a source credential for the first read-only experiment.
Its task should run `autoreconf -fi`, `./configure`, `make`, and `make check`
inside `wmcalc/` in a pinned Linux image with compiler, Autotools, `pkg-config`,
and X11/Xext/Xpm development packages. `wmcalc` defines an Autotools program
and those three `pkg-config` dependencies. It has no declared test suite, so
`make check` is initially a build smoke check; do not call it a functional
test. A later change can add focused tests.

The job should select the lab's `learning-lab` Concourse worker tag for both
resource and task steps. This tag selects a Concourse worker inside JMH's lab
VM. It is unrelated to Kubernetes Node labels. The lab has no deployment
authority, and this experiment does not deploy anything.

## Acceptance record

- [ ] Enrollment is published and a `codex` / `codex2` tsctl Job can be admitted.
- [x] A CI implementation issue is linked here.
- [x] A draft PR is linked here.
- [ ] The governed Job and independent review are linked here.
- [ ] Pipeline and task files are reviewed with a pinned build image digest.
- [x] Fly validates a temporary public Git source probe and Concourse fetches
      the exact selected branch commit.
- [ ] Fly validates the build pipeline and a temporary branch pipeline builds the
      exact PR head commit successfully.
- [ ] The fetched Git SHA, build URL, toolchain versions, and result are recorded.
- [ ] The main pipeline observes the merged commit, if a merge is authorized.
- [ ] No source credential is added for the public Git fetch; any future
      credential remains outside Git and has reviewed scope.

## Reading order

- [Decision and CI system comparison](ci-paths.md)
- [Operator log and next steps](concourse-runbook.md)
- [Existing lab demo](https://github.com/jmh-devel/ts-concourse-demo)
