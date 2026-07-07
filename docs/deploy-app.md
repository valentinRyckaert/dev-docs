# Deploying HEP Training

## Overview

This Helm chart deploys [TeSS](https://github.com/ElixirTeSS/TeSS) using a Kubernetes/OpenShift cluster.

DISCLAIMER: for the database backup, `kartoza:pg-backup:14-3.1` was not used in Kubernetes because of file system permissions issues, instead we used `pg_dump` along with a k8 cronjob that performs a backup on Monday at 2:00AM (see `templates/backup-cronjob.yaml`). To see how to access the backups see [backup section](#backup)

## Prerequisites

In order to run TeSS, you need to have the following prerequisites installed.

- git
- docker
- oc
- kubectl
- helm

These prerequisites are out of scope for this document but you can find more information about them at the following links:

- [Git](https://git-scm.com/)
- [Docker](https://www.docker.com/)
- [OpenShift CLI (oc)](https://docs.redhat.com/en/documentation/openshift_container_platform/4.2/html/cli_tools/openshift-cli-oc#cli-getting-started)
- [kubectl](https://docs.redhat.com/en/documentation/openshift_container_platform/4.2/html/cli_tools/openshift-cli-oc#usage-oc-kubectl)
- [helm](https://helm.sh/)

## Chart Structure

- `templates/`: Kubernetes manifests (Deployment, Service, PVC, etc.)
- `templates/config`: TeSS configuration files (`.env`, `tess.yml`, `secrets.yml`)
- `templates/NOTES.txt`: Post-install usage instructions
- `Chart.yaml`: Chart metadata
- `values.yaml`: Custom configuration values

## Installation

### TeSS configuration

You must already have your own [TeSS repo](https://github.com/ElixirTeSS/TeSS). Copy the `.env`, `tess.yml` and `secrets.yml` of your TeSS instance in the `templates/config` of this repo.

There are some modifications to be aware of:

1. In `.env`, Solr and Redis URL must be labelled with `-service` when deploying through Kubernetes:

```
SOLR_URL=http://solr-service:8983/solr/tess
REDIS_URL=redis://redis-service:6379/1
REDIS_TEST_URL=redis://redis-service:6379/0
```

3. In `.env`, Database host should be set up with `DB_HOST=db-service`, be sure also to have same port in `.env` and in `db.port` of `values.yaml`.

4. In `tess.yml` change `base_url` with your final URL.

### Build and push your TeSS image in a container registry

***Why***: We don't want to bake credentials in the image (i.e., leaving `.env`, `tess.yml` and `secrets.yml` in the image), instead we will setup Secrets with these three files.

We will use `ghcr.io` (GitHub Container Registry, [see documentation](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)). Your packages can be found in your GitHub Profile, `Packages` tab. You should have none or you may already have your own if you already used this service.

#### Create a token

In GitHub, go to your `Settings` > `Developer settings` (bottom of the left menu) > `Personal access tokens` > `Tokens (classic)` > `Generate new token (classic)` (top right corner).

Name your token, tick `write:packages` + `delete:packages` and create your token. [GitHub documentation](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry#authenticating-with-a-personal-access-token-classic) recommends to save it as an environment variable. In a terminal, paste your token in the command below and execute it:

    export CR_PAT=YOUR_TOKEN

#### Sign in using your token

Sign in to the Container registry, execute the command below in your terminal, change it with your GitHub username:

    echo $CR_PAT | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin

#### Build and push the app

⚠️ Make sure to have the same `.env`, `tess.yml` and `secrets.yml` in both your TeSS and TeSS-Helm-chart directories.

Execute the following command in your terminal at the root of your `TeSS` directory, do not forget to change it with your own GitHub username, the name you want to give to your image and change the tag when you re/build it (if you are on an arm64 machine, replace `linux/amd64` by `linux/arm64` on the next command):

    docker build -f Dockerfile . --build-arg CR="True" -t ghcr.io/YOUR_GITHUB_USERNAME/YOUR_IMAGE_NAME:0.1.0 --platform linux/amd64

To check that your image does not contain any credentials/sensitive information, you can run a `find` command in your image with the following command:

    docker run --rm -it ghcr.io/YOUR_GITHUB_USERNAME/YOUR_IMAGE_NAME:0.1.0 find -L \( -path "./*tess*" -o -path "./*secrets*" -o -path "./*env*" \) | grep -iw ".env\|tess.yml\|secrets.yml"

The output should only show `env.sample` and another `tess.yml` files.

Now that you are sure that neither `.env`, `tess.yml` nor `secrets.yml` are baked in your image you can push it on `ghcr.io`. Back to your terminal, you can execute this command:

    docker push ghcr.io/YOUR_GITHUB_USERNAME/YOUR_IMAGE_NAME:0.1.0

Your TeSS image is now on GitHub, without credentials and private 🎉

### Allow Kubernetes to pull your private image

If your image/package is still Private, to be able to pull it, you must configure a Secret. Execute the following command, change username, password and email with your GitHub credentials, and be at the root of the Helm chart directory:

    kubectl create secret docker-registry app-secrets-ghcr --docker-server=https://ghcr.io --docker-username=YOUR_GITHUB_USERNAME --docker-password=YOUR_GITHUB_TOKEN --docker-email=YOUR_EMAIL --dry-run=client -o yaml > templates/app-secrets-ghcr.yaml

A small explanation here. This secret is now labelled as `app-secrets-ghcr` and when any Kubernetes manifest will try to pull your image as specified under `image` it will use the secret under `imagePullSecrets`.

### Set up `.env`, `tess.yml` and `secrets.yml` as secrets

Since none of these files are baked in your image, we will set them up as Secrets:

    kubectl create secret generic app-secrets-env --from-env-file=templates/config/.env --dry-run=client -o yaml > templates/app-secrets-env.yaml
    kubectl create secret generic app-secrets-config --from-file=templates/config/secrets.yml --from-file=templates/config/tess.yml --dry-run=client -o yaml > templates/app-secrets-config.yaml

### Configure `values.yaml`

Create the `values.yaml` file:

    cp values.example.yaml values.yaml

You must change at least three fields:

- `image.repository` : change it with the image repository and change with your username and repo name
- `image.tag` : change it with the tag of your image
- `app.host` : it should be the same as `base_url` in `tess.yml` and without the `https://`
- `app.ssoRedirectUri` (if you are willing to use OIDC on TeSS) : it should be `https://YOUR_APP.HOST/users/auth/oidc/callback`

### Configure `Chart.yaml`

You can change `name`, `description` at your will. Make sure to properly change the `version` (which is your Helm chart version) and `appVersion` (which is TeSS' version).

### Deploy / Install the chart

Connect to your cluster and run:

    helm install tess .

To make changes:

    helm upgrade tess . -f values.yaml

🎉 You can now browse your TeSS instance at your specified URL.

## Uninstall

    helm uninstall tess
    
## Backup

The current deployment allows a backup on `Monday at 2:00AM`, please make sure to have sufficient storage in `backup-pvc`, by default it has 10Gi. It might become quickly saturated so be aware of this while deploying!

In order to view the backups, you can create a debug pod:

    apiVersion: v1
    kind: Pod
    metadata:
    name: debug-pod
    spec:
    containers:
    - name: shell
        image: busybox
        command: ["/bin/sh", "-c", "sleep 3600"]
        volumeMounts:
        - mountPath: /backups
        name: backup-storage
    volumes:
    - name: backup-storage
        persistentVolumeClaim:
        claimName: backup-pvc
    restartPolicy: Never

Then run these commands:

    kubectl apply -f debug-pod.yaml
    kubectl exec -it debug-pod -- ls -lh /backups
    
You should be able to see something like:

    -rw-r--r--    1 1001970000 1001970000  119.5K Jun  6 02:00 tess_20250606-020001.dump

Delete the debug-pod:

    kubectl delete pod debug-pod

## License

[BSD 3-Clause License](./LICENSE)
