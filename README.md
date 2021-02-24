# das-kustomize

Kustomize overlays for installing Styra DAS on a local minikube or crc cluster.

## Minikube
_This has been tested with DAS v0.4.6, and minikube v1.17 with Kubernetes v1.19 on a mac._

### 1. Start minikube
```
minikube start --memory=16384 --cpus=8 --driver=hyperkit

minikube ip # make note of the IP value for use later
```

### 2. Get Styra DAS configuration files 
Download the configuration files from your tenant per the Styra DAS documentation

Unzip the files to the `base` directory
```
tar xzvf on-premises.tar.gz -C base

# (Optional) Remove helm charts - you don't need them for the kustomize based install
rm -rf base/charts
```

### 3. Update config
In `overlays/minikube/kustomization.yaml`
* Update all references to `<YOUR_REGISTRY>` to your docker registry

    * _This assumes you have already loaded the Styra DAS images to your registry per the DAS on-premises documentation. If you do not have an external private registry you could [load the images directly into the minikube docker environment](https://stackoverflow.com/questions/42564058/how-to-use-local-docker-images-with-minikube)._

    * _If your private registry requires authentication, you will need to configure an [ImagePullSecret on the `default` service account](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account) in the `styra-das` namespace._

In `overlays/settings.yaml`
* Update the `ingress_url` value. Replace `<YOUR_NODE_IP>` with the value of the IP obtained in step 1.
* Update the `CHANGEME` value for the root user password.

### 4. Deploy Styra DAS
Create `styra-das` namespace
```
kubectl create ns styra-das
```

Deploy
```
kustomize build overlays/minikube | kubectl -n styra-das apply -f -
```

Open `http://<YOUR_NODE_IP>:30036` in your browser, and login in with `admin@styra.local` and the root user password defined in the prior step.

## OpenShift CodeReady Containers
_This has been tested with DAS v0.4.6, and crc v1.22 with OpenShift v4.6 on a mac._

### 1. Start crc
```
crc start --memory=18432 --cpus=8

crc ip # make note of the IP value for use later
```

### 2. Get Styra DAS configuration files 
Perform Step 2 as documented in the minikube section above.

### 3. Update config
Perform Step 3 as documented in the minikube section above.  The crc overlay uses the minikube overlay as a base, therefore the changes to the minikube files need to be made in full.

### 4. Deploy Styra DAS
Create `styra-das` project
```
oc new-project styra-das
```

Enable privileged containers in the styra-das project
```
oc adm policy add-scc-to-user privileged -z default
```

Deploy
```
kustomize build overlays/openshift-crc | kubectl -n styra-das apply -f -
```

Open `http://<YOUR_NODE_IP>:30036` in your browser, and login in with `admin@styra.local` and the root user password defined in the prior step.
