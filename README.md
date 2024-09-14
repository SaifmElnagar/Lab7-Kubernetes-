
---

# Kubernetes RBAC (Role-Based Access Control)

### What is RBAC in Kubernetes, and why is it important?
**RBAC** (Role-Based Access Control) in Kubernetes controls access to resources based on the roles assigned to users or applications. It allows fine-grained permissions to resources, enhancing security by defining who can perform what actions within the cluster.

### What are the main components of RBAC in Kubernetes?
1. **Role**: Defines a set of permissions within a namespace. It specifies which resources (like Pods, Secrets) can be accessed and the actions (like get, list, create) allowed.
2. **ClusterRole**: Similar to Role but applies to the entire cluster.
3. **RoleBinding**: Assigns a Role to a user or group within a specific namespace.
4. **ClusterRoleBinding**: Assigns a ClusterRole to users, groups, or service accounts across the entire cluster.

### How do Roles differ from ClusterRoles in Kubernetes?
- **Roles** are namespace-scoped and grant access to resources only within a specific namespace.
- **ClusterRoles** are cluster-scoped and can grant access across all namespaces or cluster-wide resources like nodes.

### What is a RoleBinding, and how does it differ from a ClusterRoleBinding?
- **RoleBinding** ties a Role to a user or service account within a namespace.
- **ClusterRoleBinding** binds a ClusterRole to a user or service account across all namespaces or the entire cluster.

### How can you list all the Roles and RoleBindings in a specific namespace?
Use the following commands:
```bash
kubectl get roles -n <namespace>
kubectl get rolebindings -n <namespace>
```

### How do you create a Role that allows a user to read secrets only in a specific namespace? (YAML Example)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: my-namespace
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]
```

### How ClusterRoleBindings grant permissions across the entire Kubernetes cluster?
ClusterRoleBindings allow a ClusterRole to apply globally, providing users or service accounts access to resources in all namespaces, or cluster-level resources like nodes.

### What are Subjects in RBAC?
Subjects represent the users, groups, or service accounts that are bound to Roles or ClusterRoles. Subjects can be:
- **User**: A human user authenticated by an external identity provider.
- **Group**: A collection of users.
- **ServiceAccount**: An account used by Pods to interact with the cluster.

### How can you check the permissions of a particular user or service account?
You can check permissions with the following command:
```bash
kubectl auth can-i <verb> <resource> --as <user> -n <namespace>
```

### What is the significance of aggregate-to-admin, aggregate-to-edit, and aggregate-to-view labels?
These labels automatically grant permissions from ClusterRoles that match those labels. It helps in providing pre-defined sets of permissions (e.g., admin, edit, view) to new ClusterRoles without redefining the roles.

---

## RBAC with External Identity Providers

### How does Kubernetes RBAC integrate with external identity providers?
Kubernetes integrates with external identity providers such as **LDAP** or **OIDC** to authenticate users. These identities are then mapped to roles and permissions within the cluster using RBAC.

### How do you troubleshoot RBAC permission errors in Kubernetes?
1. Use `kubectl auth can-i` to check what actions the user or service account can perform.
2. Review RoleBindings and ClusterRoleBindings.
3. Ensure the correct namespace is being referenced.
4. Check for typos in resource names or verbs in Roles or ClusterRoles.

### How to grant temporary elevated privileges to a user?
You can temporarily bind a high-privilege ClusterRole (e.g., admin) to a user using a ClusterRoleBinding. However, temporary elevation should be carefully managed as it increases security risks.

### Difference between RBAC and native ServiceAccount permissions for Pods?
RBAC controls access to API resources based on users or service accounts. ServiceAccount permissions, managed by RBAC, define what actions Pods can take on the cluster API. RBAC is broader and applies to all authenticated identities, while ServiceAccounts specifically control the behavior of Pods.

### Restricting access to certain Kubernetes API groups or resources using RBAC
To restrict access to specific API groups or resources, you define Roles that limit actions on certain API groups:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: restricted-ns
  name: pod-manager
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["create", "update", "delete"]
```

---

## YAML Definitions and Commands

### ClusterRole that allows listing all Pods in any namespace (YAML)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-list-clusterrole
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list"]
```

### Role to manage deployments in the dev namespace (YAML)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: deployment-manager
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["create", "update", "delete"]
```

### RoleBinding to assign the view role to john in the testing namespace (YAML)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: view-binding
  namespace: testing
subjects:
- kind: User
  name: john
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: view
  apiGroup: rbac.authorization.k8s.io
```

### ClusterRoleBinding to grant the edit role to a service account in all namespaces (YAML)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: edit-binding
subjects:
- kind: ServiceAccount
  name: developer
  namespace: default
roleRef:
  kind: ClusterRole
  name: edit
  apiGroup: rbac.authorization.k8s.io
```

### Check if a user has permission to delete Pods
```bash
kubectl auth can-i delete pods --as alice -n production
```

### Role and RoleBinding for pod-executor in the ci-cd namespace (YAML)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: ci-cd
  name: pod-executor
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create", "list", "exec"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-executor-binding
  namespace: ci-cd
subjects:
- kind: ServiceAccount
  name: pipeline-sa
  namespace: ci-cd
roleRef:
  kind: Role
  name: pod-executor
  apiGroup: rbac.authorization.k8s.io
```

### Role and RoleBinding for persistent-volume-access in the storage namespace (YAML)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: storage
  name: persistent-volume-access
rules:
- apiGroups: [""]
  resources: ["persistentvolumeclaims"]
  verbs: ["create", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pvc-access-binding
  namespace: storage
subjects:
- kind: ServiceAccount
  name: storage-admin
  namespace: storage
roleRef:
  kind: Role
  name: persistent-volume-access
  apiGroup: rbac.authorization.k8s.io
```

### ClusterRole and ClusterRoleBinding for readonly access to all resources (YAML)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: readonly-cluster
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: readonly-binding
subjects:
- kind: User
  name: alice
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: readonly-cluster
  apiGroup: rbac.authorization.k8s.io
```

---
