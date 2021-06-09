<img src="https://avatars.githubusercontent.com/u/73002963" width="300" height="300">

# Container Builder Template

This template will show you how to set up automated builds for the
Dockerfiles in your repository and deploy them to any container registry.
The example here builds to [ghcr.io/autamus/container-builder-template](https://github.com/orgs/autamus/packages/container/package/container-builder-template).

## Usage

You can either [use this repository as a template](https://github.com/autamus/container-builder-template/generate) 
or copy the [.github/workflows/build-deploy.yaml](.github/workflows/build-deploy.yaml) file 
into your current repository.

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
and credentials, usually from your CI environment:

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

The yaml recipe is documented with comments, and please 
[open an issue](https://github.com/autamus/container-builder-template/issues) if
you have any questions.

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

When you create a release (e.g., 1.0.0), the following containers will also be built.

 - ghcr.io/autamus/container-builder-template:1.0.0-latest from Dockerfile
 - ghcr.io/autamus/container-builder-template:1.0.0-subfolder from subfolder/Dockerfile

As another example, if you normally have two containers one with the tag `latest` and one 
with the tag `alpine`. This workflow step will mark them as `v0.2.1-latest` and `v0.2.1-alpine`
respectively when you create a new release with the tag with the value `v0.2.1`.

### 4. Triggers

By default, adding this workflow file will build containers on a pull request (to test
changes) and then deploy on any merge into the main branch. You can take a look
at [GitHub triggers](https://docs.github.com/en/actions/reference/events-that-trigger-workflows) if 
you want to choose a different event (e.g., a release).

#### Building on a GitHub Release

As stated above, the build-deploy template supports building containers on a new GitHub release with the trigger below.
If you decide that you'd rather not release versioned containers on a release you can delete or comment out this section.

```yaml
  # Let's also trigger a build and publish of your container when 
  # you release a new version. You can also use "created" instead of published.
  release:
    types: [published]
```

You'll also want to comment out the section that generates these different tags, shown below.

```yaml
# On a new release create a container with the same tag as the release.
- name: Set Container Tag Release
  if: github.event_name == 'release'
  run: |
    container="${{ env.registry }}/${{ env.username }}/${{ env.repository }}:${GITHUB_REF##*/}-${{ matrix.dockerfile[1] }}"
    echo "container=${container}" >> $GITHUB_ENV
```

## Interacting with GitHub Container Registry

Once your containers are deployed, you can [follow this guide](https://docs.github.com/en/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions) for how
to authenticate, pull, and otherwise interact with your personal registry. As
an example, the template here builds containers that can be pulled and interacted with:

```bash
$ docker run --rm -it ghcr.io/autamus/container-builder-template:latest
...

 Cut it out.  

          _________________  .========
         [_________________>< :======
                             '======== 
```

And for the tag:

```bash
$ docker run --rm -it ghcr.io/autamus/container-builder-template:subfolder spoon


 They say I'm rather spoontaneous!  


        __________        .-"""-.
       /          ''''---' .'    \
       \__________....---. '.    /
                          '-...-'
```
