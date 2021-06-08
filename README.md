# Container Builder Template

This template will show you how to enable automated builds for your
repository Dockerfiles to the [GitHub container registry](https://github.blog/2020-09-01-introducing-github-container-registry/), ghcr.io.

![https://avatars.githubusercontent.com/u/73002963](https://avatars.githubusercontent.com/u/73002963)

**under development**

## Usage

You can either [use this repository as a template](https://github.com/autamus/container-builder-template/template) 
or copy the contents of [.github/workflows](.github/workflows) into your current repository.

### 1. Registry

For each workflow, you'll want to define your registry in the environment of
the job, meaning filling in this section:

```yaml
# Define your registry and repository here.
# These are for the GitHub Container registry, you can also use
# Quay.io or another OCI registry
env:
  registry: docker.pkg.github.com
  username: autamus
  repository: container-builder-template     
```

For example, you might change registry to `quay.io` or similar. This is important
for naming the containers, and authenticating with the registry to push. You
will need to make these changes in both `build.yaml` and `deploy.yaml`.

### 2. Dockerfiles

To tell the workflows which Dockerfiles to build, and what tags to use, you
should define a list of `dockerfile` in the matrix. For each entry, the first
item is the relative path from the root of your repository, and the second
item is the tag to build.


```yaml
strategy:

  # A matrix of Dockerfile paths and associated tags
  # Dockerfile in root builds to tag latest
  matrix:
    dockerfiles: [[Dockerfile, latest], [subfolder/Dockerfile, subfolder]]
```

The above will build the following containers (note that docker.pkg.github.com refers
to ghcr.io)

 - ghcr.io/autamus/container-builder-template:latest from Dockerfile
 - ghcr.io/autamus/container-builder-template:subfolder from subfolder/Dockerfile


### 3. Triggers

By default, adding these files will build containers on a pull request (to test
changes) and then deploy on any merge into the main branch. You can take a look
at [GitHub triggers](https://docs.github.com/en/actions/reference/events-that-trigger-workflows) if 
you want to choose a different event (e.g., a release).

## Interacting with GitHub Container Registry

Once your containers are deployed, you can [follow this guide](https://docs.github.com/en/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions) for how
to authenticate, pull, and otherwise interact with your personal registry.
