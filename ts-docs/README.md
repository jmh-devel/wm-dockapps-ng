# Concourse CI learning trail

This directory records the first CI experiment for `jmh-devel/wm-dockapps-ng`.
The working branch is `dev/concourse-ci-lab`, based on `master` commit
`251e851a9d59d8bd0263ba1eebe08196f98f6d0b` (2026-04-12). This is a
public collection of independent, mostly Autotools based C dockapps, not one
top-level build. The first CI exercise should build one representative app,
`wmcalc`, before expanding coverage.

## Current status (2026-10-02)

1. Inspected the repository and the existing `ts-concourse-demo` lab pipeline.
2. Created `dev/concourse-ci-lab` and added the Concourse
   [pipeline](../ci/pipeline.yml) and [task](../ci/tasks/verify-wmcalc.yml).
3. The initial tsctl enrollment assumption was corrected after Joel clarified
   that this experiment runs on Concourse. The unnecessary
   [tsctl issue #1131](https://github.com/tacitness/tsctl/issues/1131) was
   closed; tsctl is not part of this CI path.
4. Fly access was restored on 2026-10-02. The existing
   `ts-concourse-demo-main/verify #2` build succeeded on `learning-lab` for
   demo commit `f1e02d1`; this verifies the lab path, not this repository.
5. GitHub Issues are disabled on this repository. The implementation is
   tracked in [ts-concourse-demo issue #3](https://github.com/jmh-devel/ts-concourse-demo/issues/3).
6. Fly validated the branch build pipeline. The first
   [Concourse `wmcalc` build](http://127.0.0.1:8080/teams/main/pipelines/wm-dockapps-ng-pr-1/jobs/verify-wmcalc/builds/1)
   succeeded on commit `7f69e00e2f5d47af248a2e996895bb137aeb3a50`.
7. The documentation was published in
   [draft PR #1](https://github.com/jmh-devel/wm-dockapps-ng/pull/1).
8. A temporary Concourse source probe validated the public HTTPS Git resource
   and fetched branch commit `ff7e4b7286cb307161ccc47a77f4adea63d81bff`
   on the `learning-lab` worker. Its
   [build #1](http://127.0.0.1:8080/teams/main/pipelines/wm-dockapps-ng-source-probe/jobs/verify-source/builds/1)
   succeeded. This is source access evidence only; no `wmcalc` build ran.

The source probe and the `wmcalc` compile build are separate evidence. The
branch pipeline fetches committed source before running the task.

## First experiment

The `verify-wmcalc` job fetches a selected Git branch using the
built-in Concourse Git resource and this public repository's HTTPS URL. The
private `ts-concourse-demo` needs a GitHub App; this public repository does not
need a source credential for the first read-only experiment.
Its task runs `autoreconf -fi`, `./configure`, `make`, and `make check`
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

- [x] A CI implementation issue is linked here.
- [x] A draft PR is linked here.
- [x] Pipeline and task files are present with a pinned base image digest.
- [x] Fly validates a temporary public Git source probe and Concourse fetches
      the exact selected branch commit.
- [x] Fly validates the build pipeline and a temporary branch pipeline builds
      commit `7f69e00e2f5d47af248a2e996895bb137aeb3a50` successfully.
- [x] The fetched Git SHA, build URL, package versions, and result are recorded.
- The final PR head build and exact SHA are recorded in
  [the PR](https://github.com/jmh-devel/wm-dockapps-ng/pull/1). Recording its
  result here would create another commit requiring another build.
- [ ] The main pipeline observes the merged commit, if a merge is authorized.
- [ ] No source credential is added for the public Git fetch; any future
      credential remains outside Git and has reviewed scope.

## Reading order

- [Decision and CI system comparison](ci-paths.md)
- [Operator log and next steps](concourse-runbook.md)
- [Concourse web UI tour](wui-tour.md)
- [Existing lab demo](https://github.com/jmh-devel/ts-concourse-demo)
