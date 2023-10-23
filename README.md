# poc-kong-and-kubernetes

Api Gateway With Kong and Kubernetes

Prerequisites:
- Kind -> Kluster Provisioning 
- Kubectl -> Kluster CLI
- Helm v3 -> Package manager --> [Install](https://helm.sh/docs/intro/install/)

> Run file kind.sh (infra/kong-k8s/kind/kind.sh)

Already in kong context run:
> kubectl get pods -A (to see all pods in the container)

Add Kong repo
> helm repo add kong https://charts.konghq.com

Install Kong using helm
> Run file kong.sh

See if Kong pods are ready:
> kubectl get pods -n kong
or
> kubectl describe pods kong-kong-{hash} -n kong

To see the logs:
> kubectl logs kong-kong-{hash} proxy -n kong

If it's necessary delete the namespace:
> kubectl delete ns kong

Adding additional tools:
> Exec -> helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
> Exec -> helm repo update
> Run file prometheus.sh (infra/kong-k8s/misc/prometheus/prometheus.sh)
> See if all pods are up -> while true; do kubectl get pods -n monitoring; sleep 0.9; done

> Exec -> helm repo add bitnami https://charts.bitnami.com/bitnami
> Exec -> helm repo update
> Run file keycloak.sh (infra/kong-k8s/misc/keycloak/keycloak.sh)

Applying services:
> Run -> kubectl create ns bets
> Run inside folder kong-k8s -> kubectl apply -f ./misc/apps --recursive -n bets
> Run -> kubectl get pods -o wide -n bets

Add ratelimit:
> Run -> kubectl apply -f misc/apis/kratelimit.yaml -n bets

Add prometheus global
> Run -> kubectl apply -f misc/apis/kprometheus.yaml

Add Ingress
> Run -> kubectl apply -f misc/apis/bets-api.yaml -n bets
> Run -> kubectl apply -f misc/apis/king.yaml -n bets

Expose keycloak
> Run -> kubectl port-forward svc/keycloak 8080:80 -n iam
> usr -> keycloak | pwd -> keycloak

Create a keycloak client called kong, enable "Client authentication" and copy the "Client secret" from "Credentials" tab. Paste 
it to file "infra/kong-k8s/misc/apis/kopenid.yaml"

> Run -> kubectl apply -f misc/apis/kopenid.yaml -n bets 

Enable plugin in bets-api.yaml "konghq.com/plugins: oidc-bets"
> kubectl apply -f misc/apis/bets-api.yaml -n bets

Create a pod to use curl
> Run -> kubectl apply -f misc/token/pod.yaml

Enter pod
> Run -> kubectl exec -it testcurl -- sh
> Run the curl command found in file infra/kong-k8s/misc/token/get-token.sh

After that make a request:

~~~
curl --location --request POST 'http://localhost/api/bets' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJJb050WF9rdXhVdzl0X1gtWkJ5RTVnSVJ2YlFlV2tjVmdpdFIzSUpFOE9RIn0.eyJleHAiOjE2OTgwMDY0ODMsImlhdCI6MTY5ODAwNjE4MywianRpIjoiZGY0OTk2YzMtNDNhNS00YWVmLThhN2ItY2M5ZDU5ZTg5MjE4IiwiaXNzIjoiaHR0cDovL2tleWNsb2FrLmlhbS9yZWFsbXMvYmV0cyIsImF1ZCI6ImFjY291bnQiLCJzdWIiOiIxNmE5MTdiNC1iNjMxLTRhOWItOGIzNy1iZTVmOTg0ZGFkYmMiLCJ0eXAiOiJCZWFyZXIiLCJhenAiOiJrb25nIiwic2Vzc2lvbl9zdGF0ZSI6IjA2ODc0ZDNlLTlhYTgtNDRjZi04N2UwLTQxZTliMDUxYmUzMCIsImFjciI6IjEiLCJyZWFsbV9hY2Nlc3MiOnsicm9sZXMiOlsib2ZmbGluZV9hY2Nlc3MiLCJ1bWFfYXV0aG9yaXphdGlvbiIsImRlZmF1bHQtcm9sZXMtYmV0cyJdfSwicmVzb3VyY2VfYWNjZXNzIjp7ImFjY291bnQiOnsicm9sZXMiOlsibWFuYWdlLWFjY291bnQiLCJtYW5hZ2UtYWNjb3VudC1saW5rcyIsInZpZXctcHJvZmlsZSJdfX0sInNjb3BlIjoib3BlbmlkIGVtYWlsIHByb2ZpbGUiLCJzaWQiOiIwNjg3NGQzZS05YWE4LTQ0Y2YtODdlMC00MWU5YjA1MWJlMzAiLCJlbWFpbF92ZXJpZmllZCI6ZmFsc2UsInByZWZlcnJlZF91c2VybmFtZSI6Im1hcmlhIiwiZ2l2ZW5fbmFtZSI6IiIsImZhbWlseV9uYW1lIjoiIn0.M7WegAZhlziFLWlf4OHcEBSfAU51Tw4lNOQpnzwo1SZVJD8HO_yDq7M3PJF5a02aGts-N840ql__w9gWAz-fTYPuMagkDgsiOKqj1maqXJueoqw-jkoy1UD52E2-xCoaC0R7vjVxFZhJEp51CqgfDNpPvw3lYjXTlaW43CmnaVtlECOkGkwf8JEdcXoGP68MyDhU3N2tir3FU2RUkvdy47twLl0MhOMBl9xn4U1XD5AHHqe2wOs-tuJeDOeRd9iNAksVfiCWevQbCp-K0b9LZB7_aC1QtFyToFoyOoEccLnrK-J5DXyAwhKaZMrC1nHP6S5IKhDq7BUPL1c3-je5jQ'
~~~