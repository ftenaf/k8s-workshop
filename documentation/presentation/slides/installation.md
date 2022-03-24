# Installation de Flux

----

<img class="r-stretch" src="../images/one-kubernetes_Sigero.png">

## La CLI

1. L'équipe **ops** récupère la _CLI_ `Flux` sur son poste de travail
1. La _CLI_ va s'appuyer sur la configuration `kubectl` pour interagir avec le _cluster_
   * connectivité réseau
   * droits _RBAC_

----

## Pré-contrôles

1. On s'assure que
   * la CLI se connecte bien
   * que les versions de `Flux` et de `Kubernetes` sont OK

```bash
$ flux check ---pre
```

----

## Le dépôt git de configuration Flux

* La CLI va créer un dépôt `fleet-infra` dans notre organisation `Github` : `one-kubernetes`
* Elle a besoin d'un _token_ `Github` capable de _CRUD_ sur les dépôts.

----

<img class="r-stretch" src="../images/github_add_token.png">

----

* `Flux` peut aussi indiquer les équipes qui auront le droit de modifier cette configuration.
* Il faut que ces équipes fassent déjà partie de l'organisation.

<img class="r-stretch" src="../images/github_teams.jpg">


----

* ⚠️ Ici on vous montre pour l'exemple, mais dans le reste du _workshop_ (comme dans la vraie vie) les différentes équipes n'ont pas besoin d'accéder à ce dépôt.  
* C'est tout l'avantage des sources de configuration multiples.

----

### Création du dépôt dans `Github`

```bash [1-3|5-10]
$ export GITHUB_TOKEN="<insert your Github personal token here>"
$ export GITHUB_USER="one-kubernetes"
$ export GITHUB_REPO="fleet-infra"

$ flux bootstrap github         \
    --owner=${GITHUB_USER}      \
    --repository=${GITHUB_REPO} \
    --team=dev1                 \
    --team=dev2                 \
    --path=clusters/snowcamp    \
```

----

### Récupération du dépôt en local

```bash
$ git clone https://github.com/${GITHUB_USER}/${GITHUB_REPO}
```

---

## Création d'une config. `staging` et `prod`

<img class="r-stretch" src="../images/one-kubernetes_Sigero.png">

* **ops** va scinder son _cluster_ en 2 : `staging` et `prod`
* Grace à _Kustomize_, elle va
  1. créer une config. de base
  2. qui sera surchargée par une config. spécifique au _tenant_
* 💡Ça paraît compliqué, mais pas d'inquiétude : la _CLI_ `Flux` s'occupe de l'essentiel

----

```bash [1|2-8]
$ cd ./fleet-infra
$ flux create kustomization tenants    \
    --namespace=flux-system            \
    --source=GitRepository/flux-system \
    --path ./tenants/staging           \
    --prune                            \
    --interval=3m                      \
    --export >> clusters/snowcamp/tenants.yaml
```

> ⚠️ N'oubliez pas de `git commit` et `git push` vers Github : c'est la source qui va être scrutée par `Flux`.

