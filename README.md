# ocm-hello-world
OCM Hello World consisting of nginx + podinfo
## Pre-requisites

- Create a kind cluster: `kind create cluster`
- Make sure flux is installed in your cluster using: `flux install`
- Install the controller using: `ocm controller install`

## Steps following 
Flags:
      --addenv                 access environment for templating
  -C, --complete               include all referenced component version
  -L, --copy-local-resources   transfer referenced local resources by-value
  -V, --copy-resources         transfer referenced resources by-value
  -c, --create                 (re)create archive
      --dry-run                evaluate and print component specifications
  -F, --file string            target file/directory (default "transport-archive")
  -f, --force                  remove existing content
  -h, --help                   help for componentversions
      --lookup stringArray     repository name or spec for closure lookup fallback
  -O, --output string          output file for dry-run
  -S, --scheme string          schema version (default "v2")
  -s, --settings stringArray   settings file with variable settings (yaml)
      --templater string       templater to use (go, none, spiff, subst) (default "subst")
  -t, --type string            archive format (directory, tar, tgz) (default "directory")
  -v, --version string         default version for components


*https://ocm.software/docs/guides/structuring-software-with-ocm/*

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm pull --destination . ingress-nginx/ingress-nginx

helm repo add podinfo https://stefanprodan.github.io/podinfo
helm pull --destination . podinfo/podinfo

# Creating componentversions adding 
ocm add componentversions --create --file ocm-hello-world --settings settings.yaml components.yaml
ocm transfer commontransportarchive ./ocm-hello-world OCIRegistry::ghcr.io/stb1337

# Creating componentarchive to add resources via cli

ocm create componentarchive github.com/stb1337/ocm-hello-world 0.0.1  --provider stb1337 --file ca-ocm-hello-world
ocm transfer componentarchive ./ca-ocm-hello-world ghcr.io/stb1337

# Setting UP cluster
https://github.com/open-component-model/ocm-website/blob/main/content/en/docs/guides/complex-component-structure-deployment.md#constructing-the-kubernetes-objects

# Inspect remote OCM 
ocm get componentversion --repo OCIRegistry::ghcr.io/stb1337/ocm-hello-world ocm.software/ocm-hello-world:0.0.7
COMPONENT                    VERSION PROVIDER
ocm.software/ocm-hello-world 0.0.7   stb1337

ocm get r --repo OCIRegistry::ghcr.io/stb1337/ocm-hello-world ocm.software/pod-info -o wide
NAME           VERSION IDENTITY TYPE      RELATION ACCESSTYPE  ACCESSSPEC
pod-info-chart 6.6.0            helmChart local    ociArtifact {"imageReference":"ghcr.io/stb1337/ocm-hello-world/ocm.software/pod-info/podinfo:6.6.0"}

## local cluster

## credentials
> https://ocm.software/docs/guides/air-gapped-gitops-with-ocm-and-flux/#gitops--localization

Then create the ServiceAccount:

cat > ./components/service_account.yaml <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: air-gapped-ops
  namespace: ocm-system
imagePullSecrets:
- name: ghcr-cred
EOF
Next, let’s modify the ComponentVersion manifest so that it points to our air-gapped OCM repository and references the ServiceAccount:

apiVersion: delivery.ocm.software/v1alpha1
kind: ComponentVersion
metadata:
  name: podinfo
  namespace: ocm-system
spec:
  interval: 1m0s
  component: phoban.io/podinfo
  version:
    semver: ">=v6.3.5"
  repository:
    url: ghcr.io/phoban01/air-gapped
  serviceAccountName: air-gapped-ops

kubectl create secret docker-registry pull-secret -n ocm-system \
    --docker-server=ghcr.io \
    --docker-username=$GITHUB_USER \
    --docker-password=$GITHUB_TOKEN \
    --docker-email=$GITHUB_USER_EMAIL

#

https://github.com/open-component-model/ocm-website/blob/main/content/en/docs/guides/complex-component-structure-deployment.md#common-issues

https://github.com/open-component-model/demo-secure-delivery/blob/main/00-setup-demo/weave-gitops/src/ocm-ctrl/config.yaml