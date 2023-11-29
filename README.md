# kubeinvaders-Istio-Datadog

Deploying kubeinvaders-repo: https://github.com/lucky-sideburn/KubeInvaders#Installation. Setting istio-ingress and monitoring the app with Datadog

## Installing kubeinvaders with helm: https://github.com/lucky-sideburn/KubeInvaders#Installation 

Installing helm repo:

```
helm repo add kubeinvaders https://lucky-sideburn.github.io/helm-charts/
helm repo update
```
Creating namespace for kubeinvaders and deploying kubeinvaders

```
helm install kubeinvaders -f invaders_values.yaml --set-string config.target_namespace="kube-system" \
-n kubeinvaders kubeinvaders/kubeinvaders
```

## Deploying istio with helm: https://istio.io/latest/docs/setup/install/helm/

Installing istio repo: 

```
helm repo add istio.io https://storage.googleapis.com/istio-release/releases/1.1.7/charts/ 
```
Creating namespace and deploying istio-base

```
helm install istio-base istio/base -n istio-system --create-namespace 
```

Deploying istiod:
```
helm install istiod istio/istiod -n istio-system --wait
```
Confirming istio deployments: 

```
kubectl get deployments -n istio-system --output wide 
```

## Deploying Istio-Ingress

Creating namespace and deploying Istio-ingress:

```
helm install istio-ingress istio/gateway -n istio-ingress --create-namespace --wait 
```
Enabling istio injection in kubeinvaders app
kubectl label namespace kubeinvaders istio-injection=enabled 

## Deploying Datadog with helm

helm install datadog-v1 -f dd_values.yaml  --set datadog.apiKey=$DD_API_KEY datadog/datadog --set targetSystem=linux -n monitoring --create-namespace --wait

## Including annotations in Istio deployment to enable Istio-Datadog integration: 

Identify istio pod example: istio-system istiod-5987b4bb4f-p8rwh

Run the edit command to update the annotations in the pod

```
kubectl edit pod istiod-5987b4bb4f-p8rwh -n istio-system
```

Then add the datadog autodiscovery annotations below: 

https://docs.datadoghq.com/integrations/istio/#configuration

```
ad.datadoghq.com/discovery.check_names: '["istio"]'
ad.datadoghq.com/discovery.init_configs: '[{}]'
ad.datadoghq.com/discovery.instances: |
     [
       {
         "istiod_endpoint": "http://%%host%%:15014/metrics",
         "use_openmetrics": "true"
       }
     ]     
```

