# Example Ansiblesque "Infrastructure as Config" DSL for Tekton

This is a strawman example of breaking down Tekton pipelines into an interface that's more familiar for
devops/sre/ops type folks.

This example uses the existing [git-clone tasks in the tekton catalog](https://github.com/tektoncd/catalog/blob/master/task/git-clone/0.2/samples/using-git-clone-result.yaml)as the first pipeline `./pipelines/validate-tag-sha`, and then fictional `ytt` and `kapp` tasks for templating and applying the kubernetes manifests `./pipelines/deploy-app` found in the git-repo provided.

Assuming the user has a CLI tool called `tkl` (tekton language) which will run this we'd see a command:

```bash
$ tkl --help
tkl is the CLI for the tkl DSL.
Usage:
  tkl [options] run.yaml

Options:
  -e --extra-vars extra variables
  -v --verbose verbose logging
  --kube-config path to kube config
  --watch wait for and watch pipelines/tasks [logs] as they run
  --dry-run do not apply (great for gitops if you just want the manifests)
  -o --output [yaml|json|human]
  --kube-context ...
  ...
  ...
```

Run pipelines directly from cli:

When you want to run the pipelines and watch them run through you can use the --watch command.  This will render out the tekton piplinerun(s) and attach to logs as the tasks run.  Omit `--watch` to let it just apply the manifests and finish.

```bash
tkl ./run.yaml --watch -e "repo-url=https://github.com/example/app" -e "expected-sha=gefrewr3"
```

Write pipeline manifests to be used by tekton:

The example output of this command is in `./output.yaml` (well the first pipeline, the second one is imaginary).

```bash
tkl ./run.yaml -e "repo-url=https://github.com/example/rep" -e "expected-sha=gefrewr3" -o yaml | tekton apply
```

## Breakdown of DSL

```
.
├── tasks
│   ├── deploy-app
│   │   ├── meta
│   │   │   └── main.yaml
│   │   ├── tasks
│   │   │   └── main.yaml
│   │   └── vars
│   │       └── main.yaml
│   └── validate-tag-sha
│       ├── meta
│       │   └── main.yaml
│       ├── tasks
│       │   └── main.yaml
│       └── vars
│           └── main.yaml
└── run.yaml
```

### run.yaml

This is the primary entrypoint, similar to the ansible `site.yaml` it lists the workspaces (kube volumes that tasks can use for staging data and passing inputs/outputs) and the task groupings to run (in order).

### tasks/validate-tag-sha

This is an example task grouping.  It has metadata, tasks, and variables, these are jammed together ( it could all be a single file, but its often useful to break it out, especially if we need to start doing some basic login in the tasks etc as well, but hopefully not, as this is focussed on `infra as config` not `infra as code`).

It calls several tasks from the tekton catalog, and a final one as defined locally. The task local could be a literal taskspec, or the shortened version using the `tkl` DSL ask shown.


### result

The end result of this is a Tekton pipelinerun manifest that delivers the tasks in order as specified in the ./tasks repo.  The resultant pipelinerun is show in `output.yaml`.