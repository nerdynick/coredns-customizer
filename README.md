# CoreDNS Customizer

CoreDNS Customiser is mostly a build script wrapping around CoreDNS an it's native build scripts to support customizing the inclusion of plugins without the need to fork and modify the core repo.

Traditionally CoreDNS wants you to fork and modify the `plugin.cfg` and build from there. 
While this can be good practise in Enterprise production deployments as you are managing your own full release cycle. This doesn't quite work for everyone, especially if you're aim is to just use an open source plugin or two.

# Usage

The included Makefile is the heart of everything. It contains a collection of targets to achieve what you might be looking for. If you don't see a final release target, please fill free to at least open an issue with details or PR with contribution.

## Building CoreDNS Executable

1. Step 1 - Define your plugins

Either fork this repo and customize or copy the Makefile and create a `plugin.cfg` file in your own project. This plugin file, `plugin.cfg`, can be treated either as an append too the CoreDNS default plugins, or a replace all default plugins.

2. Step 2 - Build the Executables

```bash
# For a full override/replace of default plugins
make build-override

# for an append to the default plugins
make build-append
```

Example with all optional variables defined with their default values
```bash
make build-append COREDNS_GIT_BRANCH=master COREDNS_GIT_REPO=https://github.com/coredns/coredns.git LINUX_ARCH="amd64 arm arm64 riscv64"
```

*Note:* The `LINUX_ARCH` variable only controls all the architecture that are built against the Linux OS. Darwin/MacOS will always be built with amd64 & arm64 and windows will always be built with amd64.

If this is all you need, then you are done and can find the built executables within the `coredns/build/{OS}/{ARCH}` folder. Otherwise continue on for all release targets you are looking for.

## Tarball and Release on Github

*NOTE:* Unlike CoreDNS's stock `MAKEFILE`, this one uses the Github CLI `gh` for all Github related actions. 
The idea being to make maintainability a bit more manageable if/when Github desides to make changes to the API over time.
This does mean that the Github CLI is a dependency of the build pipeline and will need to be installed. 
Of note, if building via the Github Actions, the selected build systems already have this installed for you out of the box.

Example using defaults and required args
```bash
make tar
make github-create-release github-upload github-publish-release GITHUB_REPO=nerdynick/coredns-customizer GITHUB_ACCESS_TOKEN={Your GH Token, can also be done as ENV Var of the same name}
```

Example with all optional variables defined with their default values
```bash
# Creating the tarballs
make tar LINUX_ARCH="amd64 arm arm64 riscv64"

# Pushing them up to Github as a Release
# All Variables are optional:
#     * COREDNS_VERSION will parse the Version from the CoreDNS Clone
#     * GITHUB_REPO will default to using the current Repo URL
make github-create-release github-upload github-publish-release COREDNS_VERSION={CoreDNS Version} CUSTOMIZED_VERSION={Version representing your changes to CoreDNS} GITHUB_REPO=nerdynick/coredns-customizer
```

## Docker Image Build and Push to DockerHub

```bash
# If building on a docker host that has a setup to use a keychain for auth. These 2 ENV vars still technically need to be defined to get a 100% successful push. However if you leave them out, everything will be pushed up but you will see additional image tags as CoreDNS uses a set of CURL calls directly to clean up manifests once it as created the 2 multi-platform manifests. 

export DOCKER_LOGIN={Your Docker Hub username}
export DOCKER_PASSWORD={Your Docker Hub password}
make docker-build docker-push DOCKER_REPO={Your Docker Repo, Usually just your Org or Username} DOCKER_NAME={Name for the Container}
```

Example with all optional variables defined with their default values
```bash
export DOCKER_LOGIN={Your Docker Hub username}
export DOCKER_PASSWORD={Your Docker Hub password}

make docker-build LINUX_ARCH="amd64 arm arm64 riscv64" DOCKER_REPO={Your Docker Repo, Usually just your Org or Username} DOCKER_NAME={Name for the Container} COREDNS_VERSION={CoreDNS Version} CUSTOMIZED_VERSION={Version representing your changes to CoreDNS}

make docker-push LINUX_ARCH="amd64 arm arm64 riscv64" DOCKER_REPO={Your Docker Repo, Usually just your Org or Username} DOCKER_NAME={Name for the Container} COREDNS_VERSION={CoreDNS Version} CUSTOMIZED_VERSION={Version representing your changes to CoreDNS}
```