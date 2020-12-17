# das-kustomize

Kustomize overlay for installing Styra DAS on a local minikube cluster.

_This has been tested with DAS v0.4.5, and minikube v1.15 with Kubernetes v1.19 on a mac._

## Start minikube
```
minikube start --memory=16384 --cpus=8 --driver=hyperkit

minikube ip # make note of the minikube ip value for use later
```

## Get Styra DAS configuration files 
Download the configuration files from your tenant per the Styra DAS documentation

Unzip the files to the `base` directory
```
tar xzvf on-premises.tar.gz -C base

# (Optional) Remove helm charts - you don't need them for the kustomize based install
rm -rf base/charts
```

## Update config
In `overlays/minikube/kustomization.yaml`
* Update all references to `<YOUR_REGISTRY>` to your docker registry

    * _This assumes you have already loaded the Styra DAS images to your registry per the DAS on-premises documentation. If you do not have an external private registry you could [load the images directly into the minikube docker environment](https://stackoverflow.com/questions/42564058/how-to-use-local-docker-images-with-minikube)._

    * _If your private registry requires authentication, you will need to configure an [ImagePullSecret on the `default` service account](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account) in the `styra-das` namespace._

In `overlays/settings.yaml`
* Update the `ingress_url` value. Replace `<YOUR_MINIKUBE_IP>` with the value from the `minikube ip` command in the prior step.
* Update the `CHANGEME` value for the root user password.

## Deploy Styra DAS
Create `styra-das` namespace
```
kubectl create ns styra-das
```

Deploy
```
kustomize build overlays/minikube | kubectl -n styra-das apply -f -
```

Open `http://<YOUR_MINIKUBE_IP>:30036` in your browser, and login in with `admin@styra.local` and the root user password defined in the prior step.
