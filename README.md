# cluster-repo-template

A template repository for a civic cloud cluster

## After cloning this template

- [ ] Switch to `cluster-template` variant specialized for your managed Kubernetes provider
    1. Check the list of available `k8s-blueprint-*` [holobranches provided by `cluster-template`](https://github.com/civic-cloud/cluster-template/tree/develop/.holo/branches) for one matching the name of your IaaS provider's managed Kubernetes solution—if none match, skip this step and continue using the default generic distribution.
    2. Edit [`.holo/branches/k8s-manifests/_cluster-template.toml`](./.holo/branches/k8s-manifests/_cluster-template.toml) and change `holosource = "=>k8s-blueprint"` to include the variant suffix
        - For example, if you're using Azure Kubernetes Service, you would change that line to `holosource = "=>k8s-blueprint-aks"`
- [ ] Customize email in 2 places in [`cert-manager.issuers.yaml`](./cert-manager.issuers.yaml)
- [ ] Customize hostname in 2 places in [`grafana/release-values.yaml`](./grafana/release-values.yaml)
- [ ] Ensure that under **Settings** / **Actions** / **General** / **Workflow permissions** for your cluster repository you have **Read and write permissions** enabled as well as **ALlow GitHub Actions to create and approve pull requests**
- [ ] Add a secret named `BOT_GITHUB_TOKEN` containing a GitHub token that can write to the repository (this is needed for a [workflow to be triggered by another workflow](https://docs.github.com/en/actions/using-workflows/triggering-a-workflow#triggering-a-workflow-from-a-workflow))
    1. Create or reuse GitHub bot user with write access to the cluster repository
    2. Grant the GitHub bot user the write access to the cluster repository, and ensure that its invitation is accepted
    3. [Create a Personal Access Token](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token) under the GitHub bot user with the `repo` and `workflow` scopes using a classic token with no expiration
    4. Save the generated Personal Access Token to a secret called `BOT_GITHUB_TOKEN` under the cluster repository
- [ ] Add a secret named `KUBECONFIG_BASE64` containing connection and authentication details for writing manifests to your Kubernetes cluster
    1. Follow IaaS provider's instructions for setting up your local `kubectl` client to administer the provisioned Kubernetes cluster
    2. Manually apply `github-actions.serviceaccount.yaml` using your local `kubectl` client to initialize the service account that GitHub Actions will use to handle deployments going forward:

        ```bash
        kubectl apply -f ./github-actions.serviceaccount.yaml
        ```

    3. Install the [`view-serviceaccount-kubeconfig`](https://github.com/superbrothers/kubectl-view-serviceaccount-kubeconfig-plugin) plugin for `kubectl` to simplify generating a `KUBECONFIG` for a service account:

        ```bash
         kubectl krew install view-serviceaccount-kubeconfig
        ```

    4. Create a token and generate a base64-encoded `KUBECONFIG` file from it:

        ```bash
        kubectl --namespace kube-system create token github-actions \
            | kubectl view-serviceaccount-kubeconfig \
            | base64
        ```

    5. Save the generated base64-encoded string to a secret called `KUBECONFIG_BASE64` under the cluster repository
