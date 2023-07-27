# spin-aks-plugin

A Spin Azure Kubernetes Service plugin with cloud provider support

## Plan

### Auth

We use the [azidentity DefaultAzureCredential](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/azidentity#NewDefaultAzureCredential) to authenticate to the Azure SDKs. This will leave users with different options for configuration that match other Azure tooling. Some of the options users will ahve include `az login` (az cli) and env variables.

### State

Stores a "state" in `$(DATA_DIR)/spin/plugins/aks/state`. This follows the Spin model described [here](https://developer.fermyon.com/spin/cache).

State that's stored includes the following

- Subscription, resource group, cluster name of most recently selected cluster
- Subscription, resource group, cluster name of most recently selected container registry
- Registry, image name, tag of most recently deployed image per application
- Last used spin aks config path

State will primarily be used to autofill prompts as intelligently as possible. This will allow most users to simply press "enter" through the prompts but also allows more advanced cases to customize usage.

### Config

A config file is where this plugin will read values from.

This file by default will be called aks-spin.toml but can be another toml file that's referenced by the global `-c` of `--config` flag. This allows for users to have different "environments".

### Azure feature selection

Users will be prompted for various Azure resources when using the cli. The CLI will provide an additional option for creating resources as well.

For example, if a user is asked for an AKS cluster we give them the option to select one but also give them the option to create a new default one.

### Versions

This plugin will be compatible with all spin versions 1.x.x

### Commands

#### spin aks init

Walks users through creating a new spin aks config.

Asks for

- config filename + location (will be autofilled as the default of `./spin-aks.toml` so most users just have to press enter)
- cluster name (subscription, rg, name)
- acr name (subscription, rg, name)
- spin.toml file location (tries to detect automatically)

#### spin aks build

Functions like spin build but also ensures that current Spin application will work for AKS (not all Spin versions are compatible, Spin version should be 1.x.x). Builds the .wasm files needed for the docker image.

#### spin aks push

User selects they are pushing to acr or another registry.

Pushes docker container to registry.

Stores the image tag in the aks spin toml config.

#### spin aks scaffold

Creates Dockerfile and manifests. Should give option of Helm, Kustomize, Kube. Dockerfile is by default output to `./Dockerfile` (with root being where the command is run). Helm output to `./charts/`, Kustomize output to `./base` and `./overlays`, and normal K8s files to `./manifests/`.

Flags

- `--dockerfile-dest` changes the destination of the Dockerfile. Filename can be included here but defaults to Dockerfile.
- `--k8s-dest` changes the destination of Kustomize, Kube, or Helm files.
- `-t or --type` chooses the type of k8s files. Options are Helm, Kustomize, and Kube.
- `--override` overrides existing files without prompts. By default the cli prompts.
- `-y` accepts all defaults meaning the cli won't prompt.
- `-c` or `--config` specifies the aks spin toml file location. Defaults to ./aks-spin.toml.

If there's already helm files or kustomize files we merge our additions with the existing files.

If we have already created these files, the cli handles updating them to the "latest versions".

Included in the manifests is a KWASM deployment along with deployments + services for the spin application.

The Dockerfile and k8s file locations are stored in the aks spin toml.

#### spin aks deploy

Applies manifests to the k8s cluster.

Doesn't support private clusters for now. We can in the future pretty easily thanks to az aks command invoke.

https://helm.sh/docs/topics/advanced/#go-sdk

https://pkg.go.dev/sigs.k8s.io/kustomize/api/krusty#Kustomizer

#### spin aks up

Goes through all steps to ensure your application is running. This is one command for all the things mentioned above.

All steps are idempotent and these commands can be used to update what's running in the cluster to a new application version.

## TODO: handle spin variables https://github.com/fermyon/spin/blob/a3d97b1aefb912ff02313875ab4f1b3c0364dac1/docs/content/sips/002-app-config.md and secrets. use annotations to seperate acr spin variables from others? Must support azure keyvault because k8s secrets are not secure.

## TODO: how to handle azure kv and acr permissions? Do we prompt to attach?

## TODO: key value store with cosmos db https://developer.fermyon.com/spin/dynamic-configuration#key-value-store-runtime-configuration

## TODO: what do we have to do for redis applications?
