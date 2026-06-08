# Ansible playbooks for the Squonk2 Viz Operator

[![lint](https://github.com/informaticsmatters/squonk2-data-manager-viz-operator-ansible/actions/workflows/lint.yaml/badge.svg)](https://github.com/informaticsmatters/squonk2-data-manager-viz-operator-ansible/actions/workflows/lint.yaml)

![GitHub](https://img.shields.io/github/license/informaticsmatters/squonk2-data-manager-viz-operator-ansible)

![GitHub tag (latest SemVer pre-release)](https://img.shields.io/github/v/tag/informaticsmatters/squonk2-data-manager-viz-operator-ansible)?include_prereleases)

[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg)](https://conventionalcommits.org)
[![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit&logoColor=white)](https://github.com/pre-commit/pre-commit)

This repo contains Ansible playbooks for the [Squonk2 Viz operator] - a
Kubernetes operator used by the **Data Manager** to create interactive data
**visualisation** instances. It is modelled on the sibling
[Squonk2 Jupyter operator] Ansible repository and deploys the **viz** operator
instead.

The operator image (`informaticsmatters/data-manager-viz-operator`) is
**public** (on DockerHub), but the visualisation instances it launches run a
**private** image (on `ghcr.io`). So, in addition to the operator, the
`site_dm` playbook deploys a **Secret** (an `ImagePullSecret`) into the Data
Manager namespace. The operator is given the *name* of that Secret through its
`SVO_IMAGE_PULL_SECRET` environment variable.

## Contributing
The project uses: -

- [pre-commit] to enforce linting of files prior to committing them to the
  upstream repository
- [Commitizen] to enforce a [Conventional Commit] commit message format

You **MUST** comply with these choices in order to  contribute to the project.

To get started review the pre-commit utility and the conventional commit style
and then set-up your local clone by following the **Installation** and
**Quick Start** sections: -

    pip install pre-commit
    pre-commit install -t commit-msg -t pre-commit

Now the project's rules will run on every commit, and you can check the
current health of your clone with: -

    pre-commit run --all-files

## Deploying into the Data Manager API
We use [Ansible] and community modules in [Ansible Galaxy] as the deployment
mechanism, using the `operator` Ansible role in this repository and a
Kubernetes config (KUBECONFIG). All of this is done via a suitable Python
environment using the requirements in the root of the project...

    python -m venv venv
    source venv/bin/activate
    pip install --upgrade pip
    pip install -r requirements.txt

Set your KUBECONFIG for the cluster and verify its right: -

    export KUBECONFIG=~/k8s-config/local-config
    kubectl get no
    [...]

Now, create a parameter file (i.e. `parameters.yaml`), setting values for the
operator that match your needs. At a minimum set the operator image tag
(`svo_image_tag`) and, for the `site_dm` playbook, the base64-encoded
image-pull secret (`svo_github_image_pull_secret`). Sensitive values such as
the image-pull secret should be supplied through an [Ansible Vault] file (a
`sensitive-<name>.vault`, selected with `svo_installation_name`), never in
plain text. Set `svo_installation_name` (e.g. to `scw-production`) so the
playbooks load the matching `sensitive-<name>.vault`; you must then provide its
vault password at run time with `--ask-vault-pass`.

Then deploy, using Ansible, from the root of the project: -

    export PARAMS=parameters
    ansible-playbook --ask-vault-pass -e @${PARAMS}.yaml site.yaml

That deploys the operator and its CRD to your chosen operator namespace
(`svo_namespace`). To deploy the Data Manager RBAC, the application
configuration and the image-pull Secret you need to run the `site_dm.yaml`
playbook: -

    ansible-playbook --ask-vault-pass -e @${PARAMS}.yaml site_dm.yaml

>   If deploying to multiple Data Managers you should just need one operator
    and then deploy the `site_dm` material to each DM namespace. Remember to
    also adjust the annotations of the CRD so each DM namespace recognises it
    as a valid application.

To remove the operator (assuming there are no operator-derived instances)...

    ansible-playbook --ask-vault-pass -e @${PARAMS}.yaml -e svo_state=absent site.yaml

>   The current Data Manager API assumes that once an Application (operator)
    has been installed it is not removed. So, removing the operator here
    is described simply to illustrate a 'clean-up' - you would not
    normally remove an Application operator in a production environment.

---

[ansible]: https://www.ansible.com
[ansible galaxy]: https://galaxy.ansible.com
[ansible vault]: https://docs.ansible.com/ansible/latest/vault_guide/index.html
[commitizen]: https://commitizen-tools.github.io/commitizen/
[conventional commit]: https://www.conventionalcommits.org/en/v1.0.0/
[pre-commit]: https://pre-commit.com
[squonk2 jupyter operator]: https://github.com/InformaticsMatters/squonk2-data-manager-jupyter-operator-ansible
[squonk2 viz operator]: https://github.com/InformaticsMatters/squonk2-data-manager-viz-operator
