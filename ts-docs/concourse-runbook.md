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
| tsctl | `tsctl repos check wm-dockapps-ng` returned `unknown repo key`. Enrollment tracked in [tsctl #1131](https://github.com/tacitness/tsctl/issues/1131). |
| GitHub issue tracker | Disabled for this public repository; implementation tracked in [ts-concourse-demo #3](https://github.com/jmh-devel/ts-concourse-demo/issues/3). |
| Fly | Login restored 2026-10-02; `fly-concourse-lab -t lab status` reports success and the `learning-lab` worker is running. |
| Lab consumer proof | `ts-concourse-demo-main/verify #2` succeeded on 2026-10-02, fetched demo commit `f1e02d1`, and ran five tests plus its Markdown link check. [Local lab build](http://127.0.0.1:8080/teams/main/pipelines/ts-concourse-demo-main/jobs/verify/builds/2). This does not validate `wm-dockapps-ng`. |
| tsctl auth dry run | `tsctl agent dispatch tsctl --runner codex --auth-profile codex2 --issue 1131 --mode implement --dry-run` summarized `codex2` but rendered `codex` metadata and legacy Secret selection. [tsctl #1123](https://github.com/tacitness/tsctl/issues/1123) tracks the defect. Live dispatch identity is unproven. |
| Managed issue queue | `tsctl agent queue ls` reports no approved server configured. This shell has neither `TSCTL_SERVER_URL` nor `TSCTL_API_KEY`; issue dispatch requires both. Do not place the API key in Git or a command argument. |
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
`ff7e4b7286cb307161ccc47a77f4adea63d81bff`. The temporary pipeline is
still present so its build history remains visible. Retire it deliberately
after the governed build pipeline supersedes it; deleting a pipeline removes
its lab build history.

## Governed sequence

1. Complete catalog enrollment and create the CI implementation tracking
   issue in the agreed location. Use runner `codex` and auth profile `codex2`
   for every tsctl dispatch.
2. Implement a small repo task and pipeline through the tsctl Job. Use the
   public HTTPS Git source without authentication. Review the build image,
   package versions, and digest; do not install packages at task runtime from
   an unpinned moving repository without recording the choice.
3. Publish a PR from `dev/concourse-ci-lab`, run an independent tsctl PR review
   Job, and resolve its findings.
4. Validate the candidate pipeline before applying it. Keep any future Fly
   vars and credentials outside Git. The existing private lab demo warns that
   `fly set-pipeline` may print resolved secret values in its diff; suppress
   that output if a later pipeline contains credentials.
5. Configure a uniquely named, temporary branch pipeline for the pushed PR
   branch. Run its verify job and record its fetched Git SHA and build URL. A
   local compile or Fly task against uncommitted files is diagnostic only.
6. After exact-head checks, independent review, and an authorized merge,
   observe a build for the merged `master` commit. Then retire the temporary
   branch pipeline deliberately, preserving the build link in this log.

## Evidence to append

| Date | PR / commit | Pipeline / build | Result | Notes |
| --- | --- | --- | --- | --- |
| 2026-10-02 | [Draft PR #1](https://github.com/jmh-devel/wm-dockapps-ng/pull/1), docs commit `901c9d2` | Demo `ts-concourse-demo-main/verify #2` | Demo passed | Target catalog, profile dry run, and managed queue access pending. |
| 2026-10-02 | Branch commit `ff7e4b7286cb307161ccc47a77f4adea63d81bff` | `wm-dockapps-ng-source-probe/verify-source #1` | Source fetch passed | Public HTTPS Git resource on `learning-lab`; no compile task. |

## Safety boundaries

This lab has no release or deployment authority. No Kubernetes application
workload is introduced. If a later change does deploy an application Pod, its
rendered manifest must positively select Nodes labeled
`node-role.kubernetes.io/worker=true` and exclude dedicated lane tolerations.
The Concourse `learning-lab` tag does not satisfy that Kubernetes rule.
