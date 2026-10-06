\# ShopHub on Kubernetes (local, GitOps)



A Flask + Postgres e-commerce API running on a local multi-node Kubernetes cluster

(kind), managed with GitOps and monitored with Prometheus/Grafana.

The same app also runs on Google Cloud Run (see the shophub-ecommerce and

shophub-infra repos for the Terraform + GitHub Actions deployment).



\## What this demonstrates

\- \*\*Cluster:\*\* kind (2 nodes) with Calico as the CNI

\- \*\*Workloads:\*\* Deployment (3 replicas, health probes), StatefulSet + PVC for Postgres, migration Job

\- \*\*GitOps:\*\* Argo CD watches this repo; pushing to `main` changes the cluster, and manual changes are reverted (self-heal)

\- \*\*Observability:\*\* kube-prometheus-stack (Prometheus + Grafana) installed with Helm

\- \*\*Security:\*\* Calico NetworkPolicies, default-deny, only the app tier can reach Postgres on 5432; secrets are created in-cluster and never committed



\## Layout

\- `kind-config.yaml`: cluster definition (default CNI disabled for Calico)

\- `apps/shophub/`: all manifests Argo CD syncs (namespace, postgres, app, migrate job, network policies)

\- `argocd/shophub-app.yaml`: the Argo CD Application



\## Proof it works

\- Scale to 3 replicas by editing `replicas` in Git → Argo scales the Deployment

\- `kubectl scale ... --replicas=1` → Argo restores the Git value

\- Unlabelled pod → `pg\_isready` returns `no response`; pod labelled `tier=app` → `accepting connections`



\## Run it yourself

1\. `kind create cluster --config kind-config.yaml --name shophub`

2\. Install Calico (tigera-operator + custom resources)

3\. Build and load the image: `kind load docker-image shophub:2.0 --name shophub`

4\. Create the `shophub-db` secret in the `shophub` namespace

5\. Install Argo CD and `kubectl apply -f argocd/shophub-app.yaml`

6\. `helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring`



\## Decisions

\- Cloud cost vs. learning: local kind keeps cost at $0; the same manifests suit GKE

\- Secrets are not stored in Git; a production setup would use Sealed Secrets or External Secrets

