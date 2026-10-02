# Concourse experiment operator log

## Observed on 2026-10-02

| Item | Evidence |
| --- | --- |
| Local source | `master` at `251e851a9d59d8bd0263ba1eebe08196f98f6d0b`; clean when inspected. |
| Experiment branch | `dev/concourse-ci-lab` created from that commit. |
| Existing CI | No `.github` workflow, Concourse pipeline, or `ts-docs/` directory was present. |
| Source access | GitHub reports `wm-dockapps-ng` as public. Use its HTTPS URL with the built-in Concourse Git resource for the initial read-only fetch; no GitHub App key is needed. |
| Candidate build | `wmcalc/configure.ac` checks `x11`, `xext`, `xpm`; `wmcalc/Makefile.am` declares `bin_PROGRAMS = wmcalc`. |
| Tests | No declared `TESTS` or `check_PROGRAMS` found for `wmcalc`; first gate is a compile smoke check. |
| Corrected assumption | I initially treated tsctl enrollment as a prerequisite. Joel clarified that this learning experiment should run on Concourse. [tsctl #1131](https://github.com/tacitness/tsctl/issues/1131) was closed as unnecessary. |
| GitHub issue tracker | Disabled for this public repository; implementation tracked in [ts-concourse-demo #3](https://github.com/jmh-devel/ts-concourse-demo/issues/3). |
| Fly | Login restored 2026-10-02; `fly-concourse-lab -t lab status` reports success and the `learning-lab` worker is running. |
| Lab consumer proof | `ts-concourse-demo-main/verify #2` succeeded on 2026-10-02, fetched demo commit `f1e02d1`, and ran five tests plus its Markdown link check. [Local lab build](http://127.0.0.1:8080/teams/main/pipelines/ts-concourse-demo-main/jobs/verify/builds/2). This does not validate `wm-dockapps-ng`. |
| Discarded tsctl path | A tsctl dry run and queue check exposed separate tsctl issues. They do not gate this Concourse experiment and no tsctl Job was dispatched. |
| Target source probe | `fly validate-pipeline` returned `looks good`; temporary pipeline `wm-dockapps-ng-source-probe/verify-source #1` succeeded on `concourse-lab-worker`. Resource version `ref` is `ff7e4b7286cb307161ccc47a77f4adea63d81bff`, matching the pushed branch at probe time. [Local lab build](http://127.0.0.1:8080/teams/main/pipelines/wm-dockapps-ng-source-probe/jobs/verify-source/builds/1). No app was compiled. |

## Source probe procedure and evidence

This temporary operator pipeline is applied in the lab, not yet versioned as
the repository's CI pipeline. It checks whether Concourse can see the public
branch and records the fetched commit without adding a source credential:

```yaml
resources:
  - name: source
    type: git
    tags: [learning-lab]
    check_every: 1m
    source:
      uri: https://github.com/jmh-devel/wm-dockapps-ng.git
      branch: dev/concourse-ci-lab

jobs:
  - name: verify-source
    plan:
      - get: source
        tags: [learning-lab]
```

The operator saved this YAML outside the checkout at
`/tmp/wm-dockapps-ng-source-probe.yml`, then ran:

```bash
fly-concourse-lab -t lab validate-pipeline -c /tmp/wm-dockapps-ng-source-probe.yml
fly-concourse-lab -t lab set-pipeline -p wm-dockapps-ng-source-probe \
  -c /tmp/wm-dockapps-ng-source-probe.yml -n
fly-concourse-lab -t lab unpause-pipeline -p wm-dockapps-ng-source-probe
fly-concourse-lab -t lab trigger-job \
  -j wm-dockapps-ng-source-probe/verify-source -w
fly-concourse-lab -t lab resource-versions \
  -r wm-dockapps-ng-source-probe/source -c 3 --json
```

Fly validation returned `looks good`. The job succeeded on 2026-10-02 and
the Git resource version reported `ref` =
`ff7e4b7286cb307161ccc47a77f4adea63d81bff`. The temporary pipeline was
paused after the successful probe and remains present so its build history is
visible without continued checks. Retire it deliberately after the versioned
build pipeline supersedes it; deleting a pipeline removes its lab build history.

## Concourse build sequence

1. Edit the versioned [pipeline](../ci/pipeline.yml) and
   [task](../ci/tasks/verify-wmcalc.yml) on `dev/concourse-ci-lab`. Use the
   public HTTPS Git source without authentication. The Debian base image is
   pinned by digest; package versions installed by Apt at task runtime are
   printed in the build log. This is a learning tradeoff, not a fully
   reproducible toolchain image.
2. Validate the pipeline with Fly and push the branch. Configure a uniquely
   named temporary branch pipeline with `-v git_branch=dev/concourse-ci-lab`.
   Run `verify-wmcalc` and record its fetched Git SHA and build URL. Concourse
   performs all compilation and `make check` work on the `learning-lab` worker.
3. Review the exact PR diff and Concourse build result. After an authorized
   merge, configure the `master` pipeline and observe a build for the merged
   commit. Pause or retire temporary branch pipelines deliberately, preserving
   build links in this log.

## Evidence to append

| Date | PR / commit | Pipeline / build | Result | Notes |
| --- | --- | --- | --- | --- |
| 2026-10-02 | [Draft PR #1](https://github.com/jmh-devel/wm-dockapps-ng/pull/1), docs commit `901c9d2` | Demo `ts-concourse-demo-main/verify #2` | Demo passed | Verified the existing lab before target work. |
| 2026-10-02 | Branch commit `ff7e4b7286cb307161ccc47a77f4adea63d81bff` | `wm-dockapps-ng-source-probe/verify-source #1` | Source fetch passed | Public HTTPS Git resource on `learning-lab`; no compile task. |

## Safety boundaries

This lab has no release or deployment authority. No Kubernetes application
workload is introduced. If a later change does deploy an application Pod, its
rendered manifest must positively select Nodes labeled
`node-role.kubernetes.io/worker=true` and exclude dedicated lane tolerations.
The Concourse `learning-lab` tag does not satisfy that Kubernetes rule.
