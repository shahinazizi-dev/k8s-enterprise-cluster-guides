
**** YOU MUST INDTALL CALICO AND TOGERA ONLY IN FIRST MASTER NODE ****
     
/// prepare images

download and tag and push images to repository ( harbor )

1 - download the required Calico images.

2 - Retag the images with the name of your registry.

3 - Push the images to your local registry.

From :  https://docs.tigera.io/calico/latest/operations/image-options/alternate-registry

=======================================================

// prepare imagers for quay.io

  pull the quay.io/tigera/opertor:v...
  
  Push the images to your registry.
  
=======================================================

/// Download tigera operator manifest file.

wget https://raw.githubusercontent.com/projectcalico/calico/v3.28.1/manifests/tigera-operator.yaml

=======================================================

/// Download the custom resources necessary to configure Calico. 

wget https://raw.githubusercontent.com/projectcalico/calico/v3.28.1/manifests/custom-resources.yaml

=======================================================

Note : ensure that tigera image version in "tigera-operator.yaml" is matche with the version of the Tigera image that was downloaded.

=======================================================

/// modify registry references to use your custom registry

sed -ie "s?quay.io?$REGISTRY?g" tigera-operator.yaml

Note : replace "$REGISTRY" with your registry address

=======================================================

/// add the image pull secret for your registry to the secret tigera-pull-secret

sed -ie "/serviceAccountName: tigera-operator/a \      imagePullSecrets:\n\      - name: $REGISTRY_PULL_SECRET"  tigera-operator.yaml


=======================================================

/// Configure "custom-resources.yaml" to use your registry for images by adding these lines to "custom-resources.yaml" in "spec:" part.

apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  registry: nodeX0101...
  calicoNetwork:
    ipPools:
    - blockSize: 26
      cidr: ....0/22

=======================================================

 **** OLD **** - OLNY IF NEEDED
 
/// create namespace - ( IF NEEDED )

 # create namespaces tigera-operator
 
 # create namespaces calico-system 

=======================================================

 **** OLD **** - OLNY IF NEEDED
 
/// create harbor secret for autetication to pull image from harbor 

#k create secret -n tigera-operator docker-registry harbor-cred --docker-username admin --docker-password xxx --docker-server opk..01.xxx --docker-email admin@example.com

=======================================================

 **** OLD **** - OLNY IF NEEDED
 
/// Configure "custom-resources.yaml" to use your registry for images by adding these lines to "custom-resources.yaml" in "spec:" part.
    the ultimate form must be  as follows. you can remove all other lines in "custom-resources.yaml" .

apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  variant: Calico
  imagePullSecrets:
    - name: tigera-pull-secret
  registry: nodeX0101...
  calicoNetwork:
    ipPools:
    - blockSize: 26
      cidr: ....0/22
      
=======================================================

Note : The pod subnet in "kubeadm-config.yaml"  and  cidr in "custom-resources.yaml" must be the same.

=======================================================

/// install tigera operator

kubectl create -f tigera-operator.yaml

 k get pod -n tigera-operator ( cheking tigera pod status )

=======================================================

/// install calico CRD 

kubectl create -f custom-resources.yaml

=======================================================

/// Verify Calico installation in your cluster.

watch kubectl get pods -n calico-system  ( checking calico pod status )

=======================================================

/// watching calico setting

k describe -n calico-system installa...

=======================================================
