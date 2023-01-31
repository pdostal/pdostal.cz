---
title: The OBSs & SCM integration
date: 2023-01-30
tags:
  - linux
---

This post will be about continuous integration between [Open Build Service] and [GitHub].

<!--more-->

## Tokens and webhooks

1) Create personal access token in GitHub with scope "repo".
     In large repositories that should be done in some service account.

2) Create token in OBS with the following parameters:
     * Type: Workflow
     * Name: GitHub
     * SCM token: GITHUB_PERSONAL_TOKEN

3) Create WebHook in the GitHub repository with the following parameters:
     * URL: https://build.opensuse.org/trigger/workflow?id=OBS_Token_ID
     * Content-Type: application/json
     * Individual events: Pull requests, Pushes

## The .obs/workflows.yml file

This file needs to go to the Git repository:

```yaml
---
pr:
  steps:
  - branch_package:
      source_project: home:pdostal
      source_package: helloworldpackage
      target_project: home:pdostal:releases
  filters:
    event: pull_request
master:
  steps:
  - trigger_services:
      project: home:pdostal
      package: helloworldpackage
  filters:
    event: push
    branches:
      only:
      - master
```

## The OBS _service file

The correspondig `_service`:

```xml
<services>
  <service name="obs_scm">
    <param name="url">https://github.com/pdostal/helloworldpackage.git</param>
    <param name="scm">git</param>
    <param name="revision">master</param>
    <param name="versionformat">@PARENT_TAG@.%h</param>
    <param name="versionrewrite-pattern">v(.*)</param>
  </service>
  <service name="set_version"/>
  <service name="tar" mode="buildtime"/>
</services>
```

The `obs_scm` service fetches the `master` from git. The package version is determined
using the `versionformat` parameter and then set to the spec file using the `ser_version`.


## Debuging:

### Github

In the Webhooks section of project settings, open the OBS webhook and then select the "Recent deliveries" tabs.
Here you see the recent triggers and their request & response bodies.

### OBS

You can see the recent events as well as it's Artifacts in "My profile" -> Tokens -> Token ID ->  Workflow Runs.

## Sources:
 * [Open Build Service Blog - Continuous Integration with OBS and GitHub/GitLab](https://openbuildservice.org/2021/05/31/scm-integration/)
 * [Open Build Service Manual -  SCM/CI Workflow Integration](https://openbuildservice.org/help/manuals/obs-user-guide/cha.obs.scm_ci_workflow_integration.html)
