# ! [🚀 GitOps with Argo Rollouts and KEDA for Scalable, Progressive Delivery](images/gitops-argo-keda-cover.png)


In this blog, I’ll walk through how I used Argo Rollouts and KEDA to implement progressive delivery and autoscaling in a Kubernetes-native way using a GitOps approach.

My GitHub repo structure looks like this:


```
gitops/
├── argo-rollouts/
│   ├── blue-green/
│   │   └── rollout.yaml
│   └── canary/
│       └── rollout.yaml
└── keda/
    └── demo-app/
        ├── templates/
        │   ├── deployment.yaml
        │   ├── service.yaml
        │   └── scaledobject.yaml
        └── values.yaml
```

# 🔁 Argo Rollouts: Progressive Delivery
# 📘 Blue-Green Deployment
Located at: gitops/argo-rollouts/blue-green/rollout.yaml

This setup creates:

A preview service (bluegreen-demo-preview)

An active service (bluegreen-demo)

A Rollout resource that controls which ReplicaSet is active/preview


```
strategy:
  blueGreen:
    autoPromotionEnabled: false
    activeService: bluegreen-demo
    previewService: bluegreen-demo-preview

```

# 🟡 Canary Deployment
Located at: gitops/argo-rollouts/canary/rollout.yaml

This rollout gradually shifts traffic to the new version based on defined steps.

```
canary:
  steps:
  - setWeight: 20
  - pause: {}
  - setWeight: 40
  - pause: {duration: 10}
  - setWeight: 60
  - pause: {duration: 10}
  - setWeight: 80
  - pause: {duration: 10}

```
Perfect for minimizing blast radius when deploying new versions.


# 📈 KEDA: Autoscaling with External Metrics
Located at: gitops/keda/demo-app/

This is a Helm chart that deploys:

A simple nginx app

A ClusterIP service

A ScaledObject that uses an external metrics-api trigger

Example ScaledObject:

```
triggers:
- type: metrics-api
  metadata:
    targetValue: "1"
    url: http://sample-app-service.demo.svc.cluster.local:8000/
    valueLocation: "place.demoes.count"

```
The app scales between 1 and 10 replicas based on the external metric response at the given URL. Super handy for custom metrics!


# 🚦 Putting It All Together with GitOps
These resources are managed via Argo CD. I push changes to the repo, and Argo CD syncs them to the cluster — this gives me:

Version-controlled delivery strategies

Reproducible environments

Instant rollback if needed

Scalable, reactive services

# 🔧 Future Improvements
Add Ingress or Gateway configuration for external access

Enable Prometheus metrics with Argo Rollouts for better visibility

Secure KEDA metric endpoints with authentication

Integrate notifications on rollout progress or failure

# 📌 Conclusion
Combining Argo Rollouts and KEDA gives a powerful GitOps-native way to manage both deployment strategies and runtime scaling. With just YAML and Git, you can automate and scale with confidence.

Check out the repo here (https://github.com/puvendhan) and feel free to fork or contribute!





