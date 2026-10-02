# Concourse experiment operator log

## Observed on 2026-10-02

| Item | Evidence |
| --- | --- |
| Local source | `master` at `251e851a9d59d8bd0263ba1eebe08196f98f6d0b`; clean when inspected. |
| Experiment branch | `dev/concourse-ci-lab` created from that commit. |
| Existing CI | No `.github` workflow, Concourse pipeline, or `ts-docs/` directory was present. |
| Candidate build | `wmcalc/configure.ac` checks `x11`, `xext`, `xpm`; `wmcalc/Makefile.am` declares `bin_PROGRAMS = wmcalc`. |
| Tests | No declared `TESTS` or `check_PROGRAMS` found for `wmcalc`; first gate is a compile smoke check. |
| tsctl | `tsctl repos check wm-dockapps-ng` returned `unknown repo key`. Enrollment tracked in [tsctl #1131](https://github.com/tacitness/tsctl/issues/1131). |
| GitHub issue tracker | Disabled for this public repository; implementation tracked in [ts-concourse-demo #3](https://github.com/jmh-devel/ts-concourse-demo/issues/3). |
| Fly | `fly-concourse-lab -t lab status` returned `please login again` / `not authorized`. |
| tsctl auth dry run | `tsctl agent dispatch tsctl --runner codex --auth-profile codex2 --issue 1131 --mode implement --dry-run` summarized `codex2` but rendered `codex` metadata and legacy Secret selection. [tsctl #1123](https://github.com/tacitness/tsctl/issues/1123) tracks the defect. Live dispatch identity is unproven. |

## Governed sequence

1. Complete catalog enrollment and create the CI implementation tracking
   issue in the agreed location. Use runner `codex` and auth profile `codex2`
   for every tsctl dispatch.
2. Implement a small repo task and pipeline through the tsctl Job. Review the
   build image, package versions, and digest; do not install packages at task
   runtime from an unpinned moving repository without recording the choice.
3. Publish a PR from `dev/concourse-ci-lab`, run an independent tsctl PR review
   Job, and resolve its findings.
4. Restore the Fly session. Validate the candidate pipeline before applying it.
   Keep the GitHub App PEM and Fly vars file outside Git. The existing lab demo
   warns that `fly set-pipeline` may print resolved secret values in its diff;
   suppress that output and inspect the exit status.
5. Configure a uniquely named, temporary branch pipeline for the pushed PR
   branch. Run its verify job and record its fetched Git SHA and build URL. A
   local compile or Fly task against uncommitted files is diagnostic only.
6. After exact-head checks, independent review, and an authorized merge,
   observe a build for the merged `master` commit. Then retire the temporary
   branch pipeline deliberately, preserving the build link in this log.

## Evidence to append

| Date | PR / commit | Pipeline / build | Result | Notes |
| --- | --- | --- | --- | --- |
| 2026-10-02 | `dev/concourse-ci-lab` at base commit `251e851` | None | Setup only | Catalog and Fly authorization pending. |

## Safety boundaries

This lab has no release or deployment authority. No Kubernetes application
workload is introduced. If a later change does deploy an application Pod, its
rendered manifest must positively select Nodes labeled
`node-role.kubernetes.io/worker=true` and exclude dedicated lane tolerations.
The Concourse `learning-lab` tag does not satisfy that Kubernetes rule.
