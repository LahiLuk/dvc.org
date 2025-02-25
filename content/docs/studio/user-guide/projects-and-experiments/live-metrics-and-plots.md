# Generate live (real-time) metrics and plots for running experiments

In your model training script, you can use [DVCLive] to send live updates for
metrics and plots without writing them to your Git repository, so that you can
track your experiments in real-time from Iterative Studio.

This requires a 2-step process:

1. [Set up an access token](#set-up-an-access-token)
2. [Send and view the updates](#send-and-view-live-metrics-and-plots)

## Set up an access token

Iterative Studio uses access tokens to authorize DVC and [DVCLive] to send live
experiment updates. The access token must be present in any request that sends
data to the Iterative Studio ingestion endpoint. Requests with missing or
incorrect access tokens are rejected with an appropriate HTTP error code and
error message. The access token is also used by DVC to notify Iterative Studio
when you push experiments using `dvc exp push`.

Once you [create your access token], pass it to your experiment. If you are
running the experiment locally, you can set the token in your [DVC config]. For
example, to set it globally for all of a user's projects:

```cli
$ dvc config --global studio.token ***
```

If you are running the experiment as part of a CI job, a secure way to provide
the access token is to create a
[GitHub secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
containing the value of the token, and use the secret in your CI job using the
`DVC_STUDIO_TOKEN` environment variable (see example below).

```yaml
steps:
  - name: Train model
    env:
      DVC_STUDIO_TOKEN: ${{ secrets.DVC_STUDIO_TOKEN }}
```

## Send and view live metrics and plots

### Send live updates using DVCLive

In the training job (which has been configured as detailed above), whenever you
log your metrics or plots using [DVCLive], they will be automatically sent to
Iterative Studio. See [DVC config] for how to enable/disable live experiment
updates and how to configure a different Studio URL or Git repository. Here is
an example of how you can use [DVCLive] in your training code:

```py
from dvclive import Live

with Live(save_dvc_exp=True) as live:
  for i in range(params["epochs"]):
    ...
    live.log_metric("accuracy", accuracy)
    live.next_step()
  ...
```

<admon type="tip">

Using `save_dvc_exp=True` will ensure that
[the results get saved as a DVC experiment](/doc/dvclive/how-it-works#track-the-results).

</admon>

<admon type="tip">

DVCLive signals the end of the experiment using `live.end()`. Using
`with Live() as live:` or one of the integrations for
[ML Frameworks](/doc/dvclive/ml-frameworks) ensures that `live.end()` is
automatically called when the experiment concludes successfully.

</admon>

### Live experiments in Iterative Studio

Iterative Studio stores the live experiments data in its database. In the
project table, the live experiments are displayed in experiment rows, which are
nested under the parent Git commit. Updates to the live experiments are
highlighted (in orange) in the project table and
[compare pane](/doc/studio/user-guide/projects-and-experiments/visualize-and-compare#compare-experiments)
in real time.

![](https://static.iterative.ai/img/studio/live_metrics.gif)

The number of live experiments with recent updates are displayed in the `Live`
icon, which can also be used to filter and show only live (running) experiments
in the table.

Live plots are displayed in the
[plots pane](/doc/studio/user-guide/projects-and-experiments/visualize-and-compare#how-to-generate-plots).
You can see them getting populated as Studio receives new updates.

![](https://static.iterative.ai/img/studio/live_plots.gif)

<admon>

If there are multiple projects connected to a single Git repository, then live
experiments for this repository are displayed in all its connected projects.

</admon>

### Detached experiments

A live experiment for which the parent Git commit is missing in the Git
repository is displayed in a separate section called `Detached experiments` at
the top of the project table.

Some of the reasons for missing parent commits are:

- the parent commit exists in your local clone of the repository and is not
  pushed to the Git remote
- the parent commit got removed by some mutative Git action such as rebase, hard
  reset with a push, squash commit, etc.

Once you push the missing parent commit to the Git remote, the live experiment
will get nested under the parent commit as expected.

You can also delete the detached experiments if they are no longer important.

### Experiment status

An experiment can have one of the following statuses:

- **Running** - Iterative Studio expects to receive live metrics and plots for
  these experiments.

  <admon type="warn">

  If the experiment stops due to any error, Iterative Studio will not be aware
  of this and it will continue to wait for live updates. In this case, you can
  delete the row from the project table.

  </admon>

- **Completed** - Iterative Studio does not expect to receive any more updates
  for these experiments. Once the experiment concludes, you can delete the row
  from the project table.

  <admon type="warn">

  Iterative Studio does not automatically commit and push the final results of
  your experiment to Git. You can [push] the experiment using appropriate DVC
  and Git commands.

  </admon>

[dvclive]: /doc/dvclive
[create your access token]:
  /doc/studio/user-guide/account-management#studio-access-token
[push]:
  /doc/user-guide/experiment-management/sharing-experiments#push-experiments
[dvc config]: /docs/user-guide/project-structure/configuration#studio
