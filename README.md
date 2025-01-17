# Setup Deploy Keys

A GitHub Action that enables the use of one or more deploy keys in order to access other private repositories. This action is based on [webfactory/ssh-agent] but has been modified to just handle deploy keys.

This modified action will take care of:

- Starting the ssh-agent
- Adding the provided keys to the agent
- Adding an [alias entry] in the ssh config for each deploy key provided.  The entry specifies which deploy key should be used for each host.
- Adding `insteadOf` rules in the git config.  These rules will rewrite git urls it encounters for the provided org/repo and point to the applicable alias entry that was added to the ssh config.  This will ensure it uses the right deploy key when interacting with different private repositories.

These entries and `insteadOf` rules are necessary when using multiple deploy keys because of the way GitHub handles requests. From [webfactory/ssh-agent]:
> When using Github deploy keys, GitHub servers will accept the first known key. But since deploy keys are scoped to a single repository, this might not be the key needed to access a particular repository. Thus, you will get the error message fatal: Could not read from remote repository. Please make sure you have the correct access rights and the repository exists. if the wrong key/repository combination is tried.

More details on context, usage, troubleshooting or known issues and limitations can be found on the original repository [webfactory/ssh-agent].

## Index <!-- omit in toc -->

- [Setup Deploy Keys](#setup-deploy-keys)
  - [Inputs](#inputs)
  - [Usage Examples](#usage-examples)
  - [Contributing](#contributing)
    - [Incrementing the Version](#incrementing-the-version)
    - [Source Code Changes](#source-code-changes)
    - [Recompiling Manually](#recompiling-manually)
    - [Updating the README.md](#updating-the-readmemd)
  - [Code of Conduct](#code-of-conduct)
  - [License](#license)
  
## Inputs

| Parameter         | Is Required | Description                                                                                                                                                                              |
|-------------------|-------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `deploy-key-info` | true        | An array of deploy key info objects that contain the org/repo that the deploy key is intended for as well as the name of the environment variable that contains the private key's value. |

## Usage Examples

```yml
name: Map project versions
on: workflow_dispatch
env: 
  # These ENV variables contain the value of the private key
  # They can be set at the workflow, job or step level
  SSH_KEY_CENTRAL_LOGGING: ${{secrets.SSH_CENTRAL_LOGGING}}
  SSH_KEY_STORAGE_ACCOUNT: ${{secrets.SSH_STORAGE_ACCOUNT}}
  SSH_KEY_ON_PREM_EGRESS: ${{secrets.SSH_ON_PREM_EGRESS}}  

jobs:
  deploy-terraform:
    runs-on: ubuntu-20.04
    steps:
        - name: Checkout Repository
          uses: actions/checkout@v3
  
        - name: Setup deploy keys for use with Terraform
          # You may also reference just the major or major.minor version
          uses: im-open/setup-deploy-keys@v1.1.3
          with:
            deploy-key-info: |
              [
                { "orgAndRepo": "im-platform/central-logging", "envName" : "SSH_KEY_CENTRAL_LOGGING" },
                { "orgAndRepo": "im-platform/storage-account-network-rules", "envName" : "SSH_KEY_STORAGE_ACCOUNT" },
                { "orgAndRepo": "im-platform/on-prem-egress-ips", "envName" : "SSH_KEY_ON_PREM_EGRESS" }
              ]
    
        - name: Setup Terraform
          id: setup
          uses: hashicorp/setup-terraform@v1.2.1
          with:
            terraform_version: ~>0.15.0
        
        - name: Terraform Init
          id: init
          run: terraform init
```

## Contributing

When creating PRs, please review the following guidelines:

- [ ] The action code does not contain sensitive information.
- [ ] At least one of the commit messages contains the appropriate `+semver:` keywords listed under [Incrementing the Version] for major and minor increments.
- [ ] The action has been recompiled.  See [Recompiling Manually] for details.
- [ ] The README.md has been updated with the latest version of the action.  See [Updating the README.md] for details.

### Incrementing the Version

This repo uses [git-version-lite] in its workflows to examine commit messages to determine whether to perform a major, minor or patch increment on merge if [source code] changes have been made.  The following table provides the fragment that should be included in a commit message to active different increment strategies.

| Increment Type | Commit Message Fragment                     |
|----------------|---------------------------------------------|
| major          | +semver:breaking                            |
| major          | +semver:major                               |
| minor          | +semver:feature                             |
| minor          | +semver:minor                               |
| patch          | *default increment type, no comment needed* |

### Source Code Changes

The files and directories that are considered source code are listed in the `files-with-code` and `dirs-with-code` arguments in both the [build-and-review-pr] and [increment-version-on-merge] workflows.  

If a PR contains source code changes, the README.md should be updated with the latest action version and the action should be recompiled.  The [build-and-review-pr] workflow will ensure these steps are performed when they are required.  The workflow will provide instructions for completing these steps if the PR Author does not initially complete them.

If a PR consists solely of non-source code changes like changes to the `README.md` or workflows under `./.github/workflows`, version updates and recompiles do not need to be performed.

### Recompiling Manually

This command utilizes [esbuild] to bundle the action and its dependencies into a single file located in the `dist` folder.  If changes are made to the action's [source code], the action must be recompiled by running the following command:

```sh
# Installs dependencies and bundles the code
npm run build
```

### Updating the README.md

If changes are made to the action's [source code], the [usage examples] section of this file should be updated with the next version of the action.  Each instance of this action should be updated.  This helps users know what the latest tag is without having to navigate to the Tags page of the repository.  See [Incrementing the Version] for details on how to determine what the next version will be or consult the first workflow run for the PR which will also calculate the next version.

## Code of Conduct

This project has adopted the [im-open's Code of Conduct](https://github.com/im-open/.github/blob/main/CODE_OF_CONDUCT.md).

## License

Copyright &copy; 2023, Extend Health, LLC. Code released under the [MIT license](LICENSE).

<!-- Links -->
[Incrementing the Version]: #incrementing-the-version
[Recompiling Manually]: #recompiling-manually
[Updating the README.md]: #updating-the-readmemd
[source code]: #source-code-changes
[usage examples]: #usage-examples
[build-and-review-pr]: ./.github/workflows/build-and-review-pr.yml
[increment-version-on-merge]: ./.github/workflows/increment-version-on-merge.yml
[esbuild]: https://esbuild.github.io/getting-started/#bundling-for-node
[git-version-lite]: https://github.com/im-open/git-version-lite
[webfactory/ssh-agent]: https://github.com/webfactory/ssh-agent
[alias entry]: https://docs.github.com/en/developers/overview/managing-deploy-keys#using-multiple-repositories-on-one-server
