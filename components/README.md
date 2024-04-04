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

