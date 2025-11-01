+++
weight = 180
+++

{{% section %}}

{{< slide template="invert" >}}

## Livraison

---

Nous sommes prêts, il est grand temps de livrer notre v1.0.

---

...mais c'est quoi l'objectif de notre livraison déjà?

---

## 🏝️ Notre livraison sera...

* Une image Docker de l'application...
* ... visible sur le [Docker Hub](https://hub.docker.com/)...
* ... avec un (Docker) tag pour chaque version

---

## 🎓 🐳 Docker Hub

Si vous n'avez pas déjà un compte sur le [Docker Hub](https://hub.docker.com/), créez-en un maintenant (nécessite une validation)

---

## 🎓 "Taguez" et déployez la version 1.0.0

- Depuis GitPod, créez un tag git local `1.0.0`
  - 💡 `git tag 1.0.0 -a -m "Première release 1.0.0, mode manuel"`

- Fabriquez l'image Docker avec le tag (Docker) 1.0.0
  - 💡 `npm run package` ?
    - npm accepte les variables d'environment!

- Publiez l'image sur le DockerHub
  - 💡 Vous devez vous authentifier avec `docker login`
  - 💡 `docker image push`
    - 💡 Peutêtre en faire un `npm run publish` ?

- Publier le tag sur votre "remote" `origin`.
  - 💡 `git push origin 1.0.0`

---

## 🎓 "Taguez" et déployez la version 1.0.0

On ajoute les nouveau scripts npm

```json
{
    "scripts": {
        "package": "docker build -t jlevesy/vehicle-server:${TAG}",
        "publish": "docker image push jlevesy/vehicle-server:${TAG}",
        "release": "npm run package && npm run publish"
    },
}
```

Ensuite pour publier notre version

```bash
docker login --username=<VOTRE USERNAME>

git tag 1.0.0 -a -m "Première release 1.0.0, mode manuel"
git push origin 1.0.0
TAG=1.0.0 npm run release
```

---

Nous avons déployé manuellement notre première image Docker, avec synchronisation historique git <-> image Docker

=> 🤔 C'était très manuel. Et si on regardait à automatiser tout ça ?

---

{{< slide template="invert" >}}

## "Continuous Everything"

---

## Livraison Continue

Continuous Delivery (CD)

---

## 🤔 Pourquoi la Livraison Continue ?

- Diminuer les risque liés au déploiement
- Permettre de récolter des retours utilisateurs plus souvent
- Rendre l'avancement visible par *tous*

---

## Qu'est ce que la Livraison Continue ?

- Suite logique de l'intégration continue:
  - Chaque changement est **potentiellement** déployable en production
  - Le déploiement peut donc être effectué à **tout** moment

*Your team prioritizes keeping the software *deployable* over working on new features*

-Martin Fowler-

---

La livraison continue est l'exercice de **mettre à disposition automatiquement** le produit logiciel pour qu'il soit prêt à être déployé à tout moment.

---

## Livraison Continue avec GitHub Actions

TL;DR; faire en sorte que sur un `git tag` notre Workflow vienne pusher l'image sur le DockerHub

---

## Prérequis: exécution conditionnelle des jobs

Il est possible d’exécuter conditionnellement un job ou un step à l'aide du mot clé `if` ([documentation](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsif))

```yaml
jobs:
  release:
    steps:
      # Lance le step dire coucou uniquement si la branche est main.
      - name: "Dire Coucou"
        run: echo "coucou"
        if: contains('refs/heads/main', github.ref)
```

---

## 🎓 🐳 Docker Hub

- Si vous n'avez pas déjà un compte sur le [Docker Hub](https://hub.docker.com/), créez-en un maintenant (nécessite une validation)
- Une fois authentifiés, naviguez dans votre compte (en haut à droite, "My Account")
- Allez dans la section "Security" et créez un nouvel "Access Token"
  - Permissions: "Read & Write" (pas besoin de "Delete")
  - ⚠️ Conservez ce token dans un endroit sûr (ne PAS partagez à d'autres)

- Insérez le token DockerHub comme secret dans votre dépôt GitHub
  - 💡 [Creating encrypted secrets for a repository](https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-a-repository)

---

## 🎓 Livraison Continue sur le DockerHub

**But :** Automatiser la publication de l'image dans le DockerHub lorsqu'un tag est poussé sur le dépot

---

- Changez votre workflow de CI de façon à ce que, sur un push de tag, les tâches suivantes soient effectuées :
  - Build, Tests et Package
  - Si c'est un tag, alors il faut créer et pousser l'image sur le DockerHub avec `npm run release`

- 💡 Il vaut s'authentifier au DockerHub avant [docker-login](https://github.com/marketplace/actions/docker-login)
- 💡 Il vous faut aussi trouver la condition a appliquer pour exécuter une étape uniquement sur un push de tag
- 💡 Ainsi que trouver le tag courant depuis le workflows
  - [La réponse est dans ce workflow](https://github.com/jlevesy/prometheus-elector/blob/main/.github/workflows/ci.yaml)

---

## ✅ Livraison Continue sur le DockerHub

```yaml
{{< snippet src="snippets/ci-docker-push-tag.yml" >}}
```

---

## Déploiement Continu

🇬🇧 Continuous Deployment / "CD"

---

## 🤔 Qu'est ce que le Déploiement Continu ?

- Version "avancée" de la livraison continue:
  - Chaque changement **est** déployé en production, de manière **automatique**

---

## Continuous Delivery VS Deployment

{{< figure src="/images/continuous-depl-vs-delivery.jpg" width=700 >}}

{{% small %}}
Source : http://blog.crisp.se/2013/02/05/yassalsundman/continuous-delivery-vs-continuous-deployment
{{% /small %}}

---

## Bénéfices du Déploiement Continu

- Rends triviale les procédures de mise en production et de rollback
  - Encourage à mettre en production le plus souvent possible
  - Encourage à faire des mises en production incrémentales
- Limite les risques d'erreur lors de la mise en production
- Fonctionne de 1 à 1000 serveurs et plus encore...

---

## 🎓 Déploiement Continu sur le DockerHub

- **But :** Déployer votre image `vehicle-server` continuellement sur le DockerHub

- Changez votre workflow de CI de façon à ce que, sur un push sur la branch `main`, les tâches suivantes soient effectuées :
  - Comme avant: on joue le cycle de vie via make.
  - SI c'est la branche `main`, alors il faut pousser l'image avec le tag `main` sur le DockerHub
  - Conservez les autre cas avec les tags

---

## 🎓 Déploiement Continu sur le DockerHub

```yaml
{{< snippet src="snippets/ci-docker-push-main.yml" >}}
```
---

## Checkpoint 🎯

- La livraison continue et le déploiement continu étendent les concepts du CI
- Les 2 sont automatisées, mais un être humain est nécessaire comme déclencheur pour la 1ère
- Le choix dépends des risques et de la "production"
- On a vu comment automatiser le déploiement dans GitHub Actions
  - Conditions dans le workflow
  - Gestion de secrets

{{% /section %}}
