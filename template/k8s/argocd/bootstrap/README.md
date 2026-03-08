## Bootstrapping ArgoCD

1. Review the branches used for deployment to the sandbox, staging and
   production environments. The default configuration will release the `develop`
   branch to the Sandbox and the `main` branch to the Production environment.
   Make sure to make the `develop` branch the default branch for PRs on a newly
   created Git repository.

   Review the branch rule in `bootstrap_root_app` job in
   `k8s/argocd/bootstrap/argocd.yaml`:

   ```
   vars:
     BRANCH:
       sh: ([ "$ENV" = "prod" ] && echo "main" || echo "develop")
   ```

   Review the `targetRevision` in the `kustomization.yaml` files shown below:

   `k8s/argocd/overlays/sandbox/apps/kustomization.yaml`:

   ```
   patches:
   - patch: |-
       - op: replace
         path: /spec/source/targetRevision
         value: develop
     target:
       kind: Application
       name: ingress
   ...
   - patch: |-
       - op: replace
         path: /spec/source/targetRevision
         value: develop
     target:
       kind: Application
       name: {{ copier__project_slug }}-prod
   ```

   `k8s/argocd/overlays/staging/apps/kustomization.yaml`:

   ```
   patches:
   - patch: |-
       - op: replace
         path: /spec/source/targetRevision
         value: develop
     target:
       kind: Application
       name: ingress
   ```

   `k8s/argocd/overlays/prod/apps/kustomization.yaml`:

   ```
   patches:
   - patch: |-
       - op: replace
         path: /spec/source/targetRevision
         value: main
     target:
       kind: Application
       name: ingress
   ```

2. Next, we need to create a GitHub deploy key to allow ArgoCD to monitor the
   repo. This step requires access to the 1password vault for your project.

   Review the vault name in the `op` cli command in
   `k8s/argocd/bootstrap/argocd.yaml`:

   ```
      - op item create
          ...
          --vault='{{ copier__project_name }}'
   ```

   Sign into 1password with `op signin` and generate the deploy key:

   ```shell
   task argocd:generate_github_deploy_key
   ```

3. Add the deploy key to your Git repository

4. Proceed with installing ArgoCD by executing:

   ```shell
   task argocd:bootstrap
   ```

The `argocd:bootstrap` task configuration is as follows:

```
  bootstrap:
      desc: Setup ArgoCD
      cmds:
        - task: install
        - task: create_repo_credentials_secret
        - task: bootstrap_root_app
```

5. ArgoCD will install the Sealed Secrets operator in the cluster. Once it is
   installed, we can generate secrets for the given environment.

   ```shell
   cd ../../..
   make debug-$ENV-secrets
   make $ENV-secrets
   ```

6. Commit the `secrets.yaml` file for the given environment and push it to the
   repo.
