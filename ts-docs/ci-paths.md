# CI path decision: Concourse, GitHub Actions, and Jenkins

## What we are learning

The goal is to observe how one Git commit becomes a build in our own Concourse
lab: resource check, version selection, task container, build log, and branch
pipeline. The first target is the `wmcalc` Autotools build. A single green
build cannot establish that every dockapp compiles or works under X11.

| Concern | Concourse lab path | GitHub Actions path | Jenkins path |
| --- | --- | --- | --- |
| Source trigger | A Git resource checks the configured branch and exposes versions; a `get` step can trigger the job. | A workflow in `.github/workflows/` uses repository events such as `push` or `pull_request`. | A Pipeline usually lives in a `Jenkinsfile`; multibranch indexing or SCM/webhook configuration discovers changes. |
| Execution definition | Pipeline YAML defines resources and jobs; a task YAML specifies image, inputs, and command. `fly set-pipeline` applies the pipeline. | Workflow YAML combines event, jobs, runners, and steps in the repo. GitHub loads it for matching events. | Declarative or scripted Groovy stages run on configured agents; controller, agents, and plugins are operated separately. |
| Worker choice | `learning-lab` is a Concourse worker tag in the JMH VM. | `runs-on` selects a GitHub hosted or self hosted runner. | `agent` or `node` selects a Jenkins executor by label. |
| Secrets | A repo scoped GitHub App key is supplied to Fly from an external vars file; Concourse stores the resolved credential. | Repository or environment secrets are injected into workflow jobs under GitHub's permission model. | Credentials are configured in Jenkins and referenced by Pipeline steps/plugins. |
| PR experience here | A temporary branch pipeline is configured manually and its exact fetched SHA is checked. This lab does not publish a GitHub required status. | PR events and status checks are native to GitHub when the workflow and permissions are configured. | Multibranch/organization jobs and GitHub integration can provide PR builds and statuses, depending on plugins and configuration. |
| Ownership cost | We run and secure the JMH Concourse VM, worker, tunnel, Fly target, Git resource, and App key. | Hosted runners reduce local infrastructure, but workflows and permissions still require maintenance and runner usage has a cost model. | We run and secure the controller, agents, credentials, and plugin set. |

## Why begin with Concourse

This repository is a learning target for an existing Concourse instance. Its
resource versions and isolated task inputs make the path from source SHA to
build explicit. A narrow smoke build exposes the older toolchain dependencies
without promising broad coverage. The pipeline stays as code in this repo;
the applied pipeline and branch variables also need an operator log because
Concourse does not infer this repository's pipeline from the file alone.

GitHub Actions would make PR triggering and GitHub status reporting simpler
for this GitHub repository. Jenkins would offer a familiar controller and
plugin based integration path, but adds controller and plugin operations to a
small first experiment. Those are tradeoffs for this lab, not judgments about
which CI system is generally best.

## Sources

- [Concourse pipelines](https://concourse-ci.org/docs/pipelines/)
- [Concourse tasks](https://concourse-ci.org/docs/tasks/)
- [GitHub Actions workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax)
- [Jenkins Pipeline](https://www.jenkins.io/doc/book/pipeline/)
