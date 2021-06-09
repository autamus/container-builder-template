<img src="https://avatars.githubusercontent.com/u/73002963" width="300" height="300">

# Container Builder Template

This template will show you how to set up automated builds for the
Dockerfiles in your repository and deploy them to any container registry.

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
  registry: ghcr.io
  username: autamus
  repository: container-builder-template     
```

Later in the workflow we need to authenticate using the same registry name,
and credentials, usually from the environment:

```yaml
- name: Log in to GitHub Docker Registry
  uses: docker/login-action@v1
  with:
    registry: ${{ env.registry }}
    username: ${{ env.username }}
    password: ${{ secrets.GITHUB_TOKEN }}
```

The above shows authentication to a GitHub Container Registry.
But if you want to use another registry like Quay.io or Docker Hub, you'll
want to provide a username and password, typically as named
secrets in the environment:

```yaml
with:
    registry: ${{ env.registry }}
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

The environment section will need to be changed in `build.yaml` and `deploy.yaml`,
while the login section is only present in `deploy.yaml`.


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

The above will build the following containers.

 - ghcr.io/autamus/container-builder-template:latest from Dockerfile
 - ghcr.io/autamus/container-builder-template:subfolder from subfolder/Dockerfile


### 4. Triggers

By default, adding these files will build containers on a pull request (to test
changes) and then deploy on any merge into the main branch. You can take a look
at [GitHub triggers](https://docs.github.com/en/actions/reference/events-that-trigger-workflows) if 
you want to choose a different event (e.g., a release).

#### Building on a GitHub Release

Although disabled by default the build-deploy template also supports building containers on a new GitHub release.
With this trigger shown below. To begin building versions of your container on a release simply uncomment the
following trigger at the beginning of the workflow.
```yaml
  # Let's also trigger a build and publish of your container when 
  # you release a new version.
  release:
    types: [published, created]
```
Now that we're building containers on GitHub releases, we'll need a way to distinguish them.
The following code segment (which is commented out by default in the workflow) will create special tags for your
docker containers to match your GitHub release.
```yaml
        # On a new release create a container with the same tag as the release.
      - name: Set Container Tag Release
        if: github.event_name == 'release'
        run: |
          container="${{ env.registry }}/${{ env.username}}/${{ env.repository }}:${GITHUB_REF##*/}-${{ matrix.dockerfile[1]}}"
          echo "container=${container}" >> $GITHUB_ENV
```
For example, let's say that you normally have two containers one with the tag `latest` and one with the tag `alpine`. This workflow step will mark them as `v0.2.1-latest` and `v0.2.1-alpine` respectively when you create a new release with the tag with the value `v0.2.1`.

## Interacting with GitHub Container Registry

Once your containers are deployed, you can [follow this guide](https://docs.github.com/en/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions) for how
to authenticate, pull, and otherwise interact with your personal registry.
