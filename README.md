# Setup
## Step 1. Make sure you have kubernetes locally
Use the [local version of Kubernetes](https://docs.docker.com/desktop/kubernetes/#enable-kubernetes) that comes with Docker desktop. See the link for installation instructions. **It can be quite resource heavy so you should disable it once you're done**.

There are other options like [kind](https://kind.sigs.k8s.io/), [minikube](https://minikube.sigs.k8s.io/docs/), [k3s](https://k3s.io/) if you prefer.

## Step 2. Connect to the cluster using kubectl
Make sure you have `kubectl` installed. It should be installed as part of Docker under `C:\Program Files\Docker\Docker\resources\bin\kubectl.exe`. If you can't type `kubectl` in a terminal window, then you need to add that path to the system environment variable called `PATH`.

- `kubectl config get-contexts` should list all the "contexts" (the clusters) you have access to. The `docker-desktop` cluster should be there
- `kubectl config use-context docker-desktop` should connect you to the local cluster
- Make sure you can access it by running `kubectl get pods -A` (list all pods in all namespaces)

## Step 3. Connect to the cluster using Lens
You might also want to connect to the cluster using Lens, so that you get a better overview of what's going on. The cluster should be listed in Lens and you should be able to connect to it.

## Step 4. Create a personal access token 
1. In order to run Flux locally, you need to create a personal access token in GitHub: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token.
1. `$env:GITHUB_TOKEN='<your token>'`
1. `$env:GITHUB_USER='<your username>'`

## Step 5. Install Flux CLI
You're going to need the Flux CLI, install it from here: https://fluxcd.io/flux/installation/#install-the-flux-cli (again, Chocolatey is an easy way for Windows).

# Simple kube apply and kustomize
## Create a namespace
1. `kubectl get namespaces` will list the existing namespaces
1. `cd manifests`
1. `kubectl apply -f namespace.yaml` will tell kubernetes to create the namespace defined in the file.
1. `kubectl get namespaces` will list the new namespace

## Create a Pod
We actually create a `Deployment`, which is responsible of keeping one or more replicas of a pod alive.

1. `kubectl apply -f deployment.yaml`
1. `kubectl get pods -n whoami` list the pods in the `whoami` namespace
1. If you delete a pod, the deployment will make sure a new replica is created. `kubectl delete pod -n whoami <name_of_pod>`

## A Service
Create a `Service` which we can use to access the pods in a load balanced way.

1. `kubectl apply -f service.yaml`
1. `kubectl get services -n whoami` to make sure the service is created. You can see that it has a `ClusterIP` but no `ExternalIP`. This means that you cannot access this service from outside the cluster.
1. A way of accessing the service is to do a port-forward. That will map a port on your local computer, to a port on the service. This isn't something you do other than in debugging purposes. `kubectl port-forward -n whoami svc/whoami 8080:80` will let you access the service from `http://localhost:8080/` on your local computer.
1. Press `ctrl+c` to abort port forwarding

## An Ingress Controller
In order to access a service (and indirectly a pod) from outside the cluster, we need an Ingress Controller. The one we use is [Traefik](https://traefik.io/).

### Setup Traefik
1. `cd traefik`
1. `kubectl apply -f 00-role.yaml -f 00-account.yaml -f 01-role-binding.yaml -f 02-traefik.yaml -f 02-traefik-services.yaml`
1. http://localhost:8080/dashboard/#/ should show you the Traefik dashboard
1. http://localhost:80 will give you a 404 (from Traefik)

### Setup an Ingress resource
1. `kubectl apply -f 03-whoami.yaml -f 03-whoami-services.yaml -f 04-whoami-ingress.yaml`
1. http://localhost:80

## Kustomize

1. Start by deleting the previous stuff `kubectl delete -f 00-role.yaml -f 00-account.yaml -f 01-role-binding.yaml -f 02-traefik.yaml -f 02-traefik-services.yaml -f 03-whoami.yaml -f 03-whoami-services.yaml -f 04-whoami-ingress.yaml`
1. `kubectl kustomize` will use kustomize to merge the files into one.
1. `kubectl apply -k`

# Doing GitOps using Flux
[Flux](https://fluxcd.io/) is a tool that helps us do "GitOps". That means that we can have a desired state in git, and it will be Flux's job to make sure that state is reconciled in the cluster.

Flux is running as a pod in the cluster, listening for changes in the git repository while also making sure the cluster matches what's in git.

1. Make sure you're good to go: `flux check --pre`
1. Install Flux in the cluster, and create a new personal reposoty flux will sync against `flux bootstrap github --owner=$env:GITHUB_USER --repository=flux-demo --branch=main --path=./clusters/my-cluster --personal`
1. Check Flux installation `kubectl get pods -n flux-system`
1. Clone your new repo: `git clone https://github.com/$env:GITHUB_USER/flux-demo` and open it in an editor
1. 