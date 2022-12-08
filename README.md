This repository contains main project and subproject.
main-project depends on subproject

To reproduce issue you need to configure the docker registry that needs auth in both `devspace.yml` files: `main-project/devspace.yaml` and `sub-project/devspace.yaml` also update both deployment images to sth existing.

```
❯ cd ../main-project
❯ devspace deploy
```

You will get output similar to:
```
❯ devspace deploy
info Using namespace 'main-project'
info Using kube context 'docker-desktop'
sub-project pullsecret:aws-ecr Ensuring image pull secret for registry: 123456789.dkr.ecr.eu-west-1.amazonaws.com...
sub-project pullsecret:aws-ecr Created image pull secret main-project/devspace-pull-secrets
sub-project deploy:sub-project Deploying chart /Users/edited/.devspace/component-chart/component-chart-0.8.5.tgz (sub-project) with helm...
sub-project deploy:sub-project Deployed helm chart (Release revision: 1)
sub-project deploy:sub-project Successfully deployed sub-project with helm
pullsecret:aws-ecr Ensuring image pull secret for registry: 123456789.dkr.ecr.eu-west-1.amazonaws.com...
deploy:main-project Deploying chart /Users/edited/.devspace/component-chart/component-chart-0.8.5.tgz (main-project) with helm...
deploy:main-project Deployed helm chart (Release revision: 1)
deploy:main-project Successfully deployed main-project with helm
```

Check the secrets for the main project:
```
❯ kubectl get secrets --namespace main-project
NAME                                    TYPE                             DATA   AGE
devspace-cache-main-project             devspace.sh/remote-cache         1      3m5s
devspace-cache-sub-project              devspace.sh/remote-cache         1      3m7s
devspace-pull-secrets                   kubernetes.io/dockerconfigjson   1      92s
sh.helm.release.v1.main-project.v1   helm.sh/release.v1               1      89s
```

And the sub-project:
```
❯ kubectl get secrets --namespace sub-project
NAME                                TYPE                 DATA   AGE
sh.helm.release.v1.sub-project.v1   helm.sh/release.v1   1      118s
If you try to deploy just the project a:
```

Now, if you deploy the subproject on its own:
```
❯ cd sub-project
❯ devspace deploy
❯ devspace deploy

warn Are you using the correct namespace?
warn Current namespace: 'sub-project'
warn Last    namespace: 'devspace'

? Which namespace do you want to use? sub-project
info Using namespace 'sub-project'
info Using kube context 'docker-desktop'
pullsecret:aws-ecr Ensuring image pull secret for registry: 123456789.dkr.ecr.eu-west-1.amazonaws.com...
pullsecret:aws-ecr Created image pull secret sub-project/devspace-pull-secrets
deploy:sub-project Deploying chart /Users/edited/.devspace/component-chart/component-chart-0.8.5.tgz (sub-project) with helm...
deploy:sub-project Deployed helm chart (Release revision: 2)
deploy:sub-project Successfully deployed sub-project with helm

```

You will see the `devspace-pull-secrets` secret was created:
```
❯ kubectl get secrets --namespace sub-project
NAME                                TYPE                             DATA   AGE
devspace-cache-sub-project          devspace.sh/remote-cache         1      35s
devspace-pull-secrets               kubernetes.io/dockerconfigjson   1      37s
sh.helm.release.v1.sub-project.v1   helm.sh/release.v1               1      3m29s
sh.helm.release.v1.sub-project.v2   helm.sh/release.v1               1      35s
```
