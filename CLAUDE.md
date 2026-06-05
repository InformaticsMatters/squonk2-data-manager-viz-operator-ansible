# CLAUDE.md

Guidance for working in this repository.

## What this is

Ansible playbooks that deploy the **Squonk2 Data Manager Viz operator** (a
Kubernetes operator) into a cluster. It is modelled on, and should keep the
same _look-and-feel_ as, the sibling
[squonk2-data-manager-jupyter-operator-ansible] repository — consult that repo
when a behaviour here is unclear.

The operator itself lives in [squonk2-data-manager-viz-operator]; it watches
`DataVisualisation` custom resources (`squonk.it/v1`, plural
`datavisualisations`) and creates a Deployment, Service and Ingress running the
private [squonk2-viz-app] image.

## Layout

- `site.yaml` — deploys the operator (CRD, Namespace, ServiceAccount, RBAC,
  Deployment) to the operator namespace (`svo_namespace`).
- `site_dm.yaml` — deploys the per-Data-Manager material (Role/RoleBinding,
  ConfigMap and the image-pull Secret) to the Data Manager namespace
  (`svo_dm_namespace`).
- `roles/operator/` — the single `operator` role.
  - `defaults/main.yaml`, `vars/main.yaml` — all variables (prefix `svo_`).
  - `tasks/` — `main` → `prep` then `deploy`/`undeploy`; `dm` → `dm-patch`.
  - `templates/*.yaml.j2` — the Kubernetes objects.
- `inventory.yaml`, `ansible.cfg` — run against `localhost`.
- `.github/workflows/lint.yaml` — CI (pre-commit + yamllint).

## Conventions (must follow)

- **Variable prefix is `svo_`** (Squonk2 Viz Operator), matching the operator's
  own `SVO_` environment-variable convention. Kubernetes object names use
  `viz-operator` (where the Jupyter repo uses `jupyter-operator`).
- **No secrets in code or logs.** The image-pull secret
  (`svo_github_image_pull_secret`) is sensitive: supply it through an Ansible
  Vault file (`sensitive-<name>.vault`, selected by `svo_installation_name`)
  or a parameters file, never in plain text. The task that deploys the Secret
  uses `no_log: true`.
- **Never let errors pass silently.** Playbooks assert their inputs (operator
  version set, DM namespace/ServiceAccount present, image-pull secret provided)
  and fail loudly when something is missing.
- Commit messages are [Conventional Commits]; `pre-commit` enforces yamllint
  and commitizen.

## Key facts

- The operator image (`svo_image`,
  `informaticsmatters/data-manager-viz-operator`) is **public** (DockerHub);
  the visualisation instance image is **private** (ghcr). The operator does not
  need the Secret, only its name, passed via the `SVO_IMAGE_PULL_SECRET`
  environment variable in `deployment.yaml.j2`.
- `svo_image_tag` defaults to `SetMe` and **must** be overridden.
- The CRD is annotated for Data Manager Application compliance, with
  `application-url-location` set to `viz.url`.
- There are no `parameters.yaml` files in the repo yet (they are gitignored);
  create one per installation when deploying.

## Local checks

    pip install pre-commit yamllint
    pre-commit run --all-files
    yamllint .
    # syntax-check the playbooks (needs ansible + kubernetes.core)
    ansible-playbook --syntax-check site.yaml site_dm.yaml

[conventional commits]: https://www.conventionalcommits.org/en/v1.0.0/
[squonk2-data-manager-jupyter-operator-ansible]: https://github.com/InformaticsMatters/squonk2-data-manager-jupyter-operator-ansible
[squonk2-data-manager-viz-operator]: https://github.com/InformaticsMatters/squonk2-data-manager-viz-operator
[squonk2-viz-app]: https://github.com/InformaticsMatters/squonk2-viz-app
