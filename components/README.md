# Components in kubernetes

## 1) Cluster Roles
      |
      └── Talks about access to a resources and permision service should have
      └── The following example shows that kubernetes can access get, list, watch from pods and nodes resources

```json
{
	"apiVersion": "rbac.authorization.k8s.io/v1",
	"kind": "ClusterRole",
	"metadata": {
		"annotations": {},
		"labels": {
			"addonmanager.kubernetes.io/mode": "Reconcile",
			"k8s-app": "kubernetes-dashboard",
			"kubernetes.io/minikube-addons": "dashboard"
		},
		"name": "kubernetes-dashboard"
	},
	"rules": [
		{
			"apiGroups": [
				"metrics.k8s.io"
			],
			"resources": [
				"pods",
				"nodes"
			],
			"verbs": [
				"get",
				"list",
				"watch"
			]
		}
	]
}
``` 

# Deployment Strategies in k8s

## 1. Recreate Deployment
   * Recreating deployment terminates all the pods and replaces them with the new version. This can be useful in situations where an old and new version of the application cannot run at the same time. The amount of downtime incurred using this strategy will depend on how long the application takes to shut down and start back up. The application state is entirely renewed since they are completely replaced.

   * An example Spec: section in the manifest file could look like this:

```bash
spec:
  replicas: 10
  strategy:
    type: Recreate
```

## 2. Rolling Deployment
   * Rolling deployments are the default K8S offering designed to reduce downtime to the cluster. A rolling deployment replaces pods running the old version of the application with the new version without downtime.

   * To achieve this, Readiness probes are used:

   * Readiness probes monitor when the application becomes available. If the probes fail, no traffic will be sent to the pod. These are used when an app needs to perform certain initialization steps before it becomes ready. An application may also become overloaded with traffic and cause the probe to fail, preventing more traffic from being sent to it and allowing it to recover.

   * Once the readiness probe detects the new version of the application is available, the old version of the application is removed. If there is a problem, the rollout can be stopped and rolled back to the previous version, avoiding downtime across the entire cluster. Because each pod is replaced one by one, deployments can take time for larger clusters. If a new deployment is triggered before another has finished, the version is updated to the version specified in the new deployment, and the previous deployment version is disregarded where it has not yet been applied.

   * A rolling deployment is triggered when something in the pod spec is changed, such as when the image, environment, or label of a pod is updated. A pod image can be updated using the command kubectl set image.

   * The spec: -> strategy: section of the manifest file can be used to refine the deployment by making use of two optional parameters — maxSurge and maxUnavailable. Both can be specified using a percentage or absolute number. A percentage figure should be used when Horizontal Pod Autoscaling is used.

   * MaxSurge specifies the maximum number of pods the Deployment is allowed to create at one time.
   * MaxUnavailable specifies the maximum number of pods that are allowed to be unavailable during the rollout.
   * For example, the configuration below would specify a requirement for 10 replicas, with a maximum of 3 being created at any one time, allowing for 1 to be unavailable during the rollout:

```bash
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 3
      maxUnavailable: 1
```

💡 You might also like:

## 3. Ramped Slow Rollout
   * Ramped Slow Rollout means the new version is rolled out slowly, creating new replicas replacing the old ones. This is also another term for a Canary deployment that we discuss later in the article.

## 4. Blue/Green Deployment
   * A Blue/Green deployment involves deploying the new application version (green) alongside the old one (blue). A load balancer in the form of the service selector object is used to direct the traffic to the new application (green) instead of the old one when it has been tested and verified. Blue/Green deployments can prove costly as twice the amount of application resources need to be stood up during the deployment period.

   * To enable this, we set up a service sitting in front of the deployments. For example, the service selector section of the manifest file for the blue deployment for an app called web-app with v1.0.0 could look like the below:

```bash
kind: Service
metadata:
 name: web-app-01
 labels:
   app: web-app
selector:
   app: web-app
   version: v1.0.0
```

   * And the deployment for the blue web app:

```bash
kind: Deployment
metadata:
  name: web-app-01
spec:
  template:
        metadata:
           labels:
             app: web-app
             version: "v1.0.0"
```

* When we want to direct traffic to the new (green) version of the app, we update the manifest to point to the new version, v2.0.0.

```bash
kind: Service
metadata:
 name: web-app-02
 labels:
   app: web-app
selector:
   app: web-app
   version: v2.0.0
```

* The deployment for the green app:

```bash
kind: Deployment
metadata:
  name: web-app-02
spec:
  template:
        metadata:
           labels:
             app: web-app
             version: "v2.0.0"
```

## 5. Best-Effort Controlled Rollout
   * Blue / green deployments may also be referred to as ‘best-effort controlled rollouts’ — again, with the focus on updating your application or microservices with a focus on minimizing downtime and ensuring that the application remains available as much as possible during the deployment process.

## 6. Shadow Deployment
   * Canary or ‘Ramped slow rollout’ strategies are sometimes used interchangeably with the term ‘Shadow Deployment’.

   * A Shadow Deployment is a strategy where a new version of an application is deployed alongside the existing production version, primarily for monitoring and testing purposes. User traffic is not actively routed to the new version in a shadow deployment.

## 7. Canary Deployments
   * A Canary deployment can be used to let a subset of the users test a new version of the application or when you are not fully confident about the new version’s functionality. This involves deploying a new version of the application alongside the old one, with the old version of the application serving most users and the newer version serving a small pool of test users. The new deployment is rolled out to more users if it is successful.

   * For example, in a K8s cluster with 100 running pods, 95 could be running v1.0.0 of the application, with 5 running the new v2.0.0 of the application. 95% of the users will be routed to the old version, and 5% will be routed to the new version. For this, we use 2 deployments side-by-side that can be scaled separately.

   * The spec section of the old application manifest would look like the following:

```bash
spec:
  replicas: 95
```

* And the new application manifest:


```bash
spec:
  replicas: 5
```
* In the example above, it might be impractical and costly to run 100 pods. A better way to achieve this is to use a load balancer such as NGINX, HAProxy, or Traefik, or a service mesh like Istio, Hashicorps Consul, or Linkrd.

* For painless and efficient maintenance of your K8s clusters, follow these 15 Kubernetes Best Practices.

## 8. A/B Deployments
   * Similar to a Canary deployment, using an A/B deployment, you can target a given subsection of users based on some target parameters (usually the HTTP headers or a cookie), as well as distribute traffic amongst versions based on weight. This technique is widely used to test the conversion of a given feature, and then the version that converts the most is rolled out.

   * This approach is usually taken based on data collected on user behavior and can be used to make better business decisions. Users are usually uninformed of the new features during the A/B testing period, so true testing can be done, and experiences between the users using the old version and those using the new version can be compared. Rollouts can be slower using A/B deployments due to the additional testing period and analysis of the user experience.

   * A/B deployments can be automated using Istio and Flagger, check out the tutorial here for more information on how to set it up

