# CKA Exam Practical Questions (Kubernetes 1.35)
## Comprehensive Question Bank - 1000+ Questions

---

# Table of Contents
1. [Cluster Architecture, Installation & Configuration (25%)](#cluster-architecture)
2. [Workloads and Scheduling (15%)](#workloads-and-scheduling)
3. [Servicing and Networking (20%)](#servicing-and-networking)
4. [Storage (10%)](#storage)
5. [Troubleshooting (30%)](#troubleshooting)

---

# CLUSTER ARCHITECTURE, INSTALLATION & CONFIGURATION (25%)

## A. RBAC (Role-Based Access Control)

### Basic RBAC Concepts

1. **Create a Role named `pod-reader` that allows reading pods in the `default` namespace.**
   - The role should have verbs: get, list, watch on pods
   - Create a RoleBinding to bind this role to a ServiceAccount named `readonly-sa`

2. **Create a ClusterRole named `node-reader` that permits reading nodes.**
   - Create a ClusterRoleBinding to bind this role to a user named `alice@example.com`

3. **Create a Role in the `production` namespace that allows:**
   - Creating, updating, deleting Deployments
   - Getting and listing Deployments
   - Name the role `deployment-manager`

4. **Create a ServiceAccount named `app-sa` in the `development` namespace.**
   - Bind the `pod-reader` ClusterRole to this ServiceAccount using a ClusterRoleBinding

5. **Create a RoleBinding in `staging` namespace that:**
   - Binds the `edit` ClusterRole to a group `developers`
   - RoleBinding name should be `developers-edit`

### Advanced RBAC Scenarios

6. **A developer named `john` needs access to:**
   - Get, list, and create Pods in `app-ns` namespace
   - Get and list Services in `app-ns` namespace
   - Cannot delete Pods or Services
   - Create appropriate Role and RoleBinding

7. **Create a ClusterRole that allows managing:**
   - Pods (all verbs)
   - Services (all verbs)
   - ConfigMaps (get, list, watch)
   - Do NOT allow editing or deleting
   - Name: `read-mostly-apps`

8. **Restrict a ServiceAccount `jenkins-sa` in `ci-cd` namespace to:**
   - Only create Pods
   - Only create ConfigMaps
   - In `ci-cd` namespace only
   - Create Role and RoleBinding

9. **Create a ClusterRole allowing access to:**
   - API groups: `apps`, `batch`, `extensions`
   - Resources: Deployments, StatefulSets, Jobs
   - All verbs (create, get, list, watch, update, patch, delete)

10. **A user needs to manage secrets in the `secure` namespace:**
    - Get, list, create, update, patch, delete Secrets
    - But NOT in other namespaces
    - Create role `secret-admin` and bind to user `admin-user`

11. **Create a role allowing:**
    - Impersonation of ServiceAccounts in the namespace
    - Used for delegation scenarios
    - Namespace: `admin-ns`

12. **Grant a ServiceAccount permission to read:**
    - ClusterRoles and ClusterRoleBindings (cluster-wide)
    - For audit purposes
    - Create ClusterRole `audit-reader`

13. **Create a role in `monitoring` namespace for Prometheus that allows:**
    - Reading all Pods (get, list, watch)
    - Reading all Services
    - Reading ConfigMaps with label `app=prometheus`

14. **A user `alice` needs temporary elevated privileges:**
    - Admin access to `test-ns` only
    - Use RoleBinding with admin ClusterRole
    - Set expiration strategy (document in comments)

15. **Create role for log aggregator that allows:**
    - Watching Pod logs (get, list, watch Pods)
    - Watching Events
    - In all namespaces
    - Use ClusterRole and ClusterRoleBinding

### RBAC Troubleshooting

16. **Identify why a ServiceAccount cannot delete Pods even though a RoleBinding exists.**
    - Hint: Check the verb in the Role

17. **A user has `admin` ClusterRole but still cannot delete resources. Debug the issue.**
    - Hint: Check ResourceQuota or NetworkPolicy

18. **Create a role that allows creating Pods but denies using privileged containers.**
    - Is this possible with RBAC alone?

### RBAC Aggregated Roles

19. **Create two ClusterRoles with label `rbac.authorization.k8s.io/aggregate-to-admin: "true"`**
    - These should automatically aggregate into the `admin` ClusterRole

20. **Create an aggregated ClusterRole for custom namespace admins:**
    - Aggregate all roles with label `app.example.com/aggregate-to-admin: "true"`

---

## B. Cluster Installation and Management with kubeadm

### Pre-Installation

21. **Prepare a Linux node for Kubernetes cluster installation.**
    - Disable swap
    - Enable IP forwarding
    - Verify kernel modules (br_netfilter, overlay)
    - Document all steps

22. **Install Container Runtime on multiple nodes:**
    - Install containerd on node1
    - Install CRI-O on node2
    - Show verification commands

23. **Install kubeadm, kubelet, kubectl on three nodes.**
    - Version: 1.35.0
    - Show version verification commands

24. **Configure kubelet to use a custom cgroup driver:**
    - Change from `cgroupfs` to `systemd`
    - Edit /etc/kubernetes/kubelet.conf

### Cluster Initialization

25. **Initialize a Kubernetes control plane with kubeadm.**
    - Set pod network CIDR: 10.244.0.0/16
    - Set service CIDR: 10.96.0.0/12
    - Output the kubeadm join command

26. **Initialize cluster and configure kubeconfig for current user.**
    - Copy admin.conf to ~/.kube/config
    - Set correct permissions

27. **Initialize cluster with custom API server flags:**
    - Set `--audit-log-path=/var/log/kubernetes/audit.log`
    - Set `--audit-policy-file=/etc/kubernetes/audit-policy.yaml`

28. **Join a worker node to the cluster.**
    - Use the kubeadm join command from control plane
    - Verify node joined successfully

29. **Join multiple worker nodes with different labels:**
    - node1: disk=ssd, type=compute
    - node2: disk=hdd, type=storage
    - Verify labels applied

### Cluster Networking

30. **Deploy Calico network plugin after cluster init.**
    - Install Calico
    - Verify all nodes are Ready
    - Verify all pods are running

31. **Deploy Flannel instead of Calico.**
    - Install and configure Flannel
    - Verify pods can communicate across nodes

32. **Deploy custom CNI plugin:**
    - Configure cluster without default CNI
    - Manually install custom plugin via manifest

33. **Verify CNI plugin is working correctly.**
    - Check pod network CIDR
    - Test pod-to-pod communication
    - Show network interface configuration

### Upgrading and Managing Cluster

34. **Upgrade Kubernetes cluster from 1.34 to 1.35.**
    - Drain control plane node
    - Upgrade kubeadm, kubelet, kubectl
    - Perform drain and uncordon
    - Upgrade worker nodes
    - Document all steps

35. **Perform a rolling upgrade of worker nodes.**
    - Drain node
    - Upgrade kubeadm, kubelet, kubectl
    - Uncordon and verify
    - Repeat for each node

36. **Backup etcd before cluster upgrade.**
    - Create snapshot backup
    - Restore from backup
    - Verify restore succeeded

37. **Restore etcd from backup.**
    - Stop apiserver
    - Restore etcd database
    - Verify cluster recovered

38. **Backup all cluster configurations.**
    - Backup kubeadm-config ConfigMap
    - Backup static manifests
    - Backup certificates

### Cluster Certificates

39. **Check certificate expiration dates on control plane.**
    - Use kubeadm certs check-expiration
    - Identify certificates expiring soon

40. **Renew cluster certificates before expiration.**
    - Use kubeadm certs renew all
    - Restart affected components

41. **Rotate kubelet certificate manually.**
    - Find certificate path
    - Generate new certificate
    - Update kubelet configuration

42. **Create custom certificate for API server.**
    - Generate CA
    - Generate server certificate
    - Configure API server to use new certificate

### Static Manifests and Control Plane

43. **Deploy control plane components using static manifests.**
    - Move manifests to /etc/kubernetes/manifests/
    - Verify components are running

44. **Create a custom static Pod that runs on all nodes.**
    - Deploy monitoring agent as static Pod
    - Verify it's running on all nodes

45. **Modify control plane component arguments:**
    - Edit kube-apiserver manifest
    - Add custom audit flags
    - Verify changes applied

### Multi-Control Plane Setup

46. **Set up a highly available control plane with 3 masters.**
    - Configure etcd clustering
    - Setup load balancer for API servers
    - Join second and third masters

47. **Configure external etcd cluster.**
    - Setup 3-node etcd cluster
    - Configure Kubernetes to use external etcd
    - Verify cluster health

48. **Perform maintenance on single master in HA setup.**
    - Drain master node
    - Upgrade components
    - Rejoin cluster

49. **Recover failed master node in HA cluster.**
    - Remove failed node
    - Initialize new master with kubeadm
    - Join to cluster

50. **Configure load balancer for multiple API servers.**
    - Setup HAProxy or NGINX
    - Point all kubelets to load balancer
    - Verify high availability

---

## C. Extension Interfaces (CNI, CSI, CRI)

### CNI (Container Network Interface)

51. **Understand CNI plugin architecture and installation.**
    - Research how CNI plugins are called
    - Manual test of CNI binary

52. **Configure custom CNI plugin configuration file.**
    - Create CNI config in /etc/cni/net.d/
    - Specify network CIDR and gateway
    - Test pod network connectivity

53. **Debug pod network connectivity issues.**
    - Check CNI logs
    - Verify CNI binary permissions
    - Test with direct kubectl commands

54. **Deploy multiple CNI plugins in cluster.**
    - Flannel for pod networking
    - Cilium for network policies
    - Configure to work together

55. **Switch CNI plugin from Flannel to Calico.**
    - Remove Flannel
    - Install Calico
    - Verify existing pods still work

### CRI (Container Runtime Interface)

56. **Configure different container runtimes:**
    - containerd
    - CRI-O
    - Docker (legacy)
    - Show which runtime is configured

57. **Check which CRI is in use by kubelet.**
    - Check kubelet logs
    - Inspect kubelet config
    - Verify via cri socket

58. **Switch container runtime from Docker to containerd.**
    - Drain node
    - Install containerd
    - Update kubelet CRI socket path
    - Restart kubelet
    - Uncordon and verify

59. **Manage container images with custom CRI runtime.**
    - Use crictl to inspect images
    - Remove unused images
    - Verify disk space

60. **Configure CRI plugin options:**
    - containerd: configure registry mirrors
    - CRI-O: configure storage driver
    - Show configuration files

### CSI (Container Storage Interface)

61. **Install CSI driver for external storage.**
    - Install NFS CSI driver
    - Verify driver pod running
    - Create StorageClass using CSI driver

62. **Create PVC using CSI-based StorageClass.**
    - Create CSI StorageClass
    - Create PVC
    - Deploy pod using PVC
    - Verify storage mounted

63. **Configure CSI driver to provision volumes automatically.**
    - Setup dynamic provisioning
    - Create test PVC
    - Verify volume provisioned

64. **Understand CSI driver architecture.**
    - Identify DaemonSet for node plugin
    - Identify Deployment for controller plugin
    - Show how CSI driver communicates with kubelet

65. **Troubleshoot CSI driver issues.**
    - Check driver logs
    - Verify RBAC permissions
    - Check if driver is registered

---

## D. CRDs and Operators

### Custom Resource Definitions (CRDs)

66. **Create a Custom Resource Definition (CRD):**
    - Name: `Database`
    - Group: `database.example.com`
    - Version: v1
    - Define schema with fields: engine, version, size

67. **Create multiple versions of a CRD:**
    - v1alpha1
    - v1beta1
    - v1
    - Show storage version handling

68. **Create Custom Resources based on the CRD.**
    - Create PostgreSQL resource
    - Create MySQL resource
    - Verify they are stored in etcd

69. **Add validation rules to CRD schema.**
    - Validate `size` is between 1-100
    - Validate `engine` is one of: postgres, mysql, mariadb
    - Test with invalid resource creation

70. **Implement CRD with RBAC permissions.**
    - Create role to allow managing custom resources
    - Create role to allow only reading custom resources
    - Verify permissions work

### Operators

71. **Install and use a known operator (e.g., Prometheus Operator).**
    - Install operator
    - Create custom resource to configure Prometheus
    - Verify operator creates all necessary resources

72. **Install another operator (e.g., EDB Postgres Operator).**
    - Deploy operator
    - Create Postgres cluster custom resource
    - Verify Postgres pods created and running

73. **Manage operator lifecycle.**
    - Update operator to newer version
    - Handle backward compatibility
    - Rollback if needed

74. **Create a simple custom operator:**
    - Define CRD for application
    - Write controller logic (Python or Go)
    - Deploy as Deployment
    - Test operator creates expected resources

75. **Debug operator issues.**
    - Check operator logs
    - Verify RBAC permissions for operator
    - Check if custom resources are being processed

---

## E. Helm and Kustomize

### Helm Basics

76. **Install Helm 3.**
    - Verify installation
    - Setup shell completion

77. **Add Helm repository and search for charts.**
    - Add bitnami repository
    - Search for nginx chart
    - Show chart details

78. **Install a Helm chart (e.g., Nginx).**
    - Install nginx chart from bitnami repo
    - Set release name and namespace
    - Expose via service

79. **Install chart with custom values.**
    - Deploy PostgreSQL with custom values
    - Set image.tag, persistence.size, replicas
    - Verify resources created with custom values

80. **Upgrade Helm release.**
    - Upgrade to newer chart version
    - Update values in upgrade
    - Verify upgrade completed successfully

81. **Rollback Helm release to previous version.**
    - Identify previous revision
    - Perform rollback
    - Verify resources reverted

82. **Uninstall Helm release.**
    - Remove release completely
    - Verify all resources deleted
    - Keep namespace

### Helm Chart Creation

83. **Create a Helm chart from scratch.**
    - Use `helm create myapp`
    - Understand chart structure
    - Deploy created chart

84. **Create chart with custom templates.**
    - Create Deployment template with values
    - Create Service template
    - Create ConfigMap template
    - Test template rendering

85. **Use chart dependencies.**
    - Add dependency: nginx chart
    - Create Chart.yaml with dependencies
    - Run `helm dependency update`
    - Install parent chart

86. **Create chart with sub-charts.**
    - Create parent chart with multiple dependencies
    - Each dependency has its own values
    - Install and verify all resources

87. **Add custom resource templates to chart.**
    - Create CRD template
    - Create custom resource template
    - Package and deploy

### Helm Advanced Topics

88. **Package Helm chart for distribution.**
    - Run `helm package myapp`
    - Create chart repository
    - Upload chart

89. **Create custom Helm repository.**
    - Setup HTTP server
    - Create index.yaml
    - Add repository to helm client
    - Install chart from custom repo

90. **Debug Helm chart issues.**
    - Use `helm template` to inspect rendered manifests
    - Use `helm lint` to validate chart
    - Use `helm get values` to see deployed values

91. **Conditional rendering in Helm templates.**
    - Use `if` to conditionally include resources
    - Use `range` to loop through values
    - Deploy chart with different values

### Kustomize Basics

92. **Create a Kustomization base.**
    - Create kustomization.yaml
    - Define Deployment, Service, ConfigMap
    - Deploy using kubectl apply -k

93. **Create Kustomize overlays for different environments.**
    - dev overlay: replicas=1
    - prod overlay: replicas=3
    - Test deploying each overlay

94. **Use Kustomize patches to modify resources.**
    - Patch Deployment replicas
    - Patch Service type
    - Deploy and verify

95. **Use Kustomize namespace transformer.**
    - Set namespace for all resources in overlay
    - Deploy to specific namespace

96. **Use Kustomize labels and annotations.**
    - Add common labels to all resources
    - Add common annotations
    - Verify applied to resources

### Kustomize Advanced Topics

97. **Use Kustomize var replacement.**
    - Define variables in kustomization.yaml
    - Reference variables in resources
    - Deploy and verify variables substituted

98. **Create complex Kustomize structure.**
    - Base with core resources
    - Multiple overlays for different environments
    - Each overlay modifies base differently
    - Deploy complete structure

99. **Use Kustomize with generators.**
    - Generate ConfigMap from file
    - Generate Secret from literal
    - Reference in Deployment

100. **Combine Kustomize with Helm.**
     - Use Helm templates in Kustomize base
     - Apply Kustomize patches on top
     - Deploy final result

---

# WORKLOADS AND SCHEDULING (15%)

## A. Deployments and Rolling Updates

### Basic Deployments

101. **Create a basic Deployment:**
     - 3 replicas of nginx:latest
     - Set resource limits
     - Label appropriately

102. **Scale Deployment to different replica counts.**
     - Scale to 5 replicas
     - Scale to 1 replica
     - Verify pods created/terminated correctly

103. **Update Deployment image.**
     - Update from nginx:1.14 to nginx:1.20
     - Use kubectl set image
     - Watch rollout progress

104. **Perform rolling update with custom strategy.**
     - Set maxSurge: 1
     - Set maxUnavailable: 1
     - Update image and observe behavior

105. **View Deployment rollout history.**
     - Show all revisions
     - Show details of specific revision
     - Identify which revision is current

### Rolling Updates and Rollbacks

106. **Rollback Deployment to previous version.**
     - Identify issue with current version
     - Rollback to previous revision
     - Verify pods updated with old image

107. **Rollback to specific revision.**
     - View rollout history
     - Rollback to revision 2
     - Verify correct revision deployed

108. **Pause Deployment rollout.**
     - Update image (triggers rollout)
     - Pause rollout while in progress
     - Resume and complete rollout

109. **Monitor Deployment status during update.**
     - Watch replica availability
     - Check for errors in pods
     - Understand what "ready", "up-to-date", "available" mean

110. **Create Deployment with readiness probe to control rollout.**
     - Add readiness probe
     - Perform update
     - Observe how readiness probe affects rollout

### Advanced Deployment Strategies

111. **Create Canary Deployment:**
     - Deploy 90% traffic to v1 (9 replicas)
     - Deploy 10% traffic to v2 (1 replica)
     - Gradually shift traffic (show in comments)

112. **Create Blue-Green Deployment:**
     - Blue: v1 running (3 replicas)
     - Green: v2 running (3 replicas)
     - Switch traffic between them using Service selector

113. **Create Deployment with custom preStop lifecycle hook.**
     - Add preStop hook to gracefully shutdown
     - Trigger pod deletion
     - Observe hook execution in logs

114. **Create Deployment with postStart lifecycle hook.**
     - Add postStart hook to setup configuration
     - Trigger pod creation
     - Verify hook executed

115. **Use init containers in Deployment.**
     - Init container to download config
     - Main container uses config
     - Verify init runs before main

---

## B. StatefulSets

116. **Create a StatefulSet:**
     - 3 replicas
     - Headless service for network identity
     - Redis pods with persistent storage

117. **Scale StatefulSet up and down.**
     - Scale from 3 to 5 replicas
     - Observe ordered creation
     - Scale down to 1
     - Observe ordered termination

118. **Update StatefulSet image.**
     - Reverse rolling update strategy
     - Update image
     - Observe update starts from last replica

119. **Understand ordinal naming in StatefulSet.**
     - View pod names (pod-0, pod-1, pod-2)
     - Understand network identity relationships
     - Verify DNS names

120. **Create StatefulSet with volumeClaimTemplates.**
     - Define PVC template
     - Deploy StatefulSet
     - Verify each pod gets its own PVC
     - Verify data persistence

121. **Perform surgery on StatefulSet.**
     - Delete single pod (e.g., pod-1)
     - Verify it's recreated with same ordinal
     - Verify data preserved

122. **Test StatefulSet pod affinity.**
     - Configure antiAffinity
     - Deploy across nodes
     - Scale beyond available nodes
     - Verify pending pods

---

## C. DaemonSets

123. **Create a DaemonSet:**
     - Deploy monitoring agent on all nodes
     - Verify pods on every node

124. **Create DaemonSet with node selector.**
     - Label nodes: disk-type=ssd, disk-type=hdd
     - Deploy DaemonSet only to ssd nodes
     - Verify pods only on matching nodes

125. **Create DaemonSet with tolerations.**
     - Deploy to tainted nodes (e.g., control plane)
     - Add toleration for control plane taint
     - Verify pods on all nodes including masters

126. **Update DaemonSet image.**
     - Update to new image version
     - Observe rolling update behavior
     - Understand updateStrategy options

127. **Configure DaemonSet rolling update strategy.**
     - Set maxUnavailable to control update pace
     - Update image
     - Observe one node updated at a time

---

## D. Jobs and CronJobs

### Jobs

128. **Create a simple Job.**
     - Run one-time task (e.g., database migration)
     - Job completes successfully

129. **Create Job with multiple completions.**
     - Completions: 3 (run 3 times)
     - Watch job execution
     - Verify all completions succeeded

130. **Create Job with parallelism.**
     - Parallelism: 2 (run 2 in parallel)
     - Completions: 4
     - Verify 2 pods run at once

131. **Create Job with backoff limit.**
     - Set backoffLimit: 3
     - Make job fail on purpose
     - Verify job retries 3 times then fails

132. **Create Job with activeDeadlineSeconds.**
     - Set activeDeadlineSeconds: 30
     - Job will timeout if running longer
     - Verify job terminates after timeout

133. **Create Job with restartPolicy: Never.**
     - Job fails
     - Observe new pod created instead of restart
     - Verify failed pod still exists for inspection

134. **Create Job with restartPolicy: OnFailure.**
     - Job fails
     - Observe pod restarted instead of new pod created

### CronJobs

135. **Create a CronJob.**
     - Run every hour
     - Execute backup script
     - Schedule: `0 * * * *`

136. **Create CronJob with specific schedule.**
     - Run at 2:30 AM daily
     - Schedule: `30 2 * * *`

137. **Create CronJob with concurrency policy.**
     - concurrencyPolicy: Forbid
     - If previous job still running, skip execution
     - Test by creating long-running job

138. **Create CronJob with history limits.**
     - successfulJobsHistoryLimit: 3
     - failedJobsHistoryLimit: 2
     - Keep only last 3 successful, 2 failed jobs

139. **Update CronJob schedule.**
     - Change schedule to run every 30 minutes
     - Verify new schedule takes effect for next run

140. **Manually trigger CronJob.**
     - Use kubectl create job to manually run job
     - Verify job executes immediately

---

## E. ConfigMaps and Secrets

### ConfigMaps

141. **Create ConfigMap from literal values.**
     - kubectl create configmap with key=value pairs
     - Verify ConfigMap created

142. **Create ConfigMap from file.**
     - kubectl create configmap from configuration file
     - Verify file contents in ConfigMap

143. **Create ConfigMap from directory.**
     - kubectl create configmap from directory
     - Multiple files in directory
     - Each file becomes key

144. **Use ConfigMap as environment variables in Pod.**
     - Mount ConfigMap as env vars
     - Deploy pod
     - Execute `env` command to verify

145. **Use ConfigMap as mounted volume in Pod.**
     - Mount ConfigMap as volume
     - Verify files appear in container

146. **Update ConfigMap and observe pod effects.**
     - Update ConfigMap key value
     - Verify mounted volume updated
     - Verify env var NOT updated (demonstrates difference)

147. **Create immutable ConfigMap.**
     - Set immutable: true
     - Try to update
     - Verify update fails

### Secrets

148. **Create Secret from literal values.**
     - Create Secret with username, password
     - Verify Secret created

149. **Create Secret from file.**
     - Create Secret from certificate file
     - Create Secret from key file

150. **Create TLS Secret for HTTPS.**
     - Create TLS cert and key
     - Create TLS Secret
     - Use in Ingress

151. **Use Secret as environment variables.**
     - Mount Secret as env vars
     - Verify variables available in pod
     - Verify values base64 decoded

152. **Use Secret as mounted volume.**
     - Mount Secret as volume
     - Verify files appear in container

153. **Create Docker registry Secret.**
     - Create Secret for private image registry
     - Use in Pod imagePullSecrets
     - Deploy pod using private image

154. **Create generic Secret and decode values.**
     - Create Secret with encoded value
     - Extract Secret yaml
     - Base64 decode the value

155. **Verify Secret is encrypted at rest.**
     - Check encryption configuration
     - Verify secret data is encrypted in etcd

---

## F. Pod Autoscaling

### Horizontal Pod Autoscaler (HPA)

156. **Create basic HPA for Deployment.**
     - Min replicas: 2, Max: 10
     - Target CPU utilization: 80%
     - Deploy test load to trigger scaling

157. **Create HPA based on memory utilization.**
     - Target memory: 1000Mi
     - Observe scaling as memory increases

158. **Create HPA with custom metrics.**
     - Use metrics-server
     - Query custom application metric
     - Scale based on custom metric

159. **Create HPA with multiple metrics.**
     - Scale if CPU > 70% OR Memory > 80%
     - Test each condition triggers scaling

160. **View HPA status and metrics.**
     - kubectl get hpa
     - kubectl describe hpa
     - Understand current vs target values

161. **Modify HPA min/max replicas.**
     - Update HPA to new limits
     - Verify takes effect

### Vertical Pod Autoscaler (VPA)

162. **Install and configure VPA.**
     - Deploy VPA components
     - Verify running in cluster

163. **Create VPA for Deployment.**
     - Mode: Auto (automatically updates resources)
     - Deploy with initial requests/limits
     - VPA adjusts based on actual usage

164. **Use VPA in recommendation mode.**
     - Mode: Off (only provides recommendations)
     - Review recommendations
     - Manually apply if desired

---

## G. Pod Scheduling and Affinity

### Node Affinity

165. **Create Pod with nodeSelector.**
     - Label node: disk=ssd
     - Create Pod that only runs on ssd nodes
     - Verify pod scheduled on correct node

166. **Create Pod with required node affinity.**
     - Must run on nodes with label: zone=us-west
     - Create pod
     - If no matching node, pod stays pending

167. **Create Pod with preferred node affinity.**
     - Prefer nodes with label: gpu=true
     - If no gpu nodes, schedule on any node
     - Test with and without matching nodes

168. **Schedule Pod on specific node by hostname.**
     - Create pod that must run on node1
     - Use nodeAffinity with hostname requirement

169. **Create Pod with multiple node affinity terms.**
     - Prefer: SSD nodes
     - Prefer: East region
     - Demonstrate weight-based preference

### Pod Affinity and AntiAffinity

170. **Create Pod with required pod affinity.**
     - Pod must run on same node as app=db pod
     - Deploy pod
     - Verify it's on same node as database pod

171. **Create Pod with preferred pod affinity.**
     - Pod prefers same node as app=cache pod
     - If no cache pod exists, schedule on any node

172. **Create Pod with required pod antiaffinity.**
     - Pod cannot run on same node as app=frontend pod
     - Deploy 3 replicas
     - Verify spread across nodes

173. **Create Pod antiaffinity across availability zones.**
     - topologyKey: topology.kubernetes.io/zone
     - Pod cannot be in same zone as other replicas
     - Verify spread across zones

174. **Test pod affinity with pending pods.**
     - Create affinity requiring pod not yet scheduled
     - First pod stays pending
     - Once requirement pod scheduled, first pod schedules

### Taints and Tolerations

175. **Add taint to node.**
     - Taint: key=gpu, effect=NoSchedule
     - Create pod without toleration
     - Verify pod doesn't schedule on tainted node

176. **Create Pod with toleration.**
     - Add toleration for gpu taint
     - Deploy pod
     - Verify pod schedules on tainted node

177. **Add taint to node with effect=NoExecute.**
     - Taint existing pods with effect=NoExecute
     - Watch pods evicted from node
     - Pods with toleration remain

178. **Use toleration with tolerationSeconds.**
     - Add toleration with tolerationSeconds: 300
     - Pod allowed to stay 5 minutes after taint added
     - After 5 minutes, pod evicted

179. **Add multiple taints to single node.**
     - Taint1: gpu=true, NoSchedule
     - Taint2: highmem=true, NoSchedule
     - Create pod with both tolerations
     - Pod schedules successfully

180. **Prevent pod eviction on tainted node.**
     - Add NoExecute taint
     - Create pod with toleration but no tolerationSeconds
     - Pod not evicted, stays indefinitely

### Resource Requests and Limits

181. **Create Pod with resource requests.**
     - Request: CPU 100m, Memory 128Mi
     - Deploy pod
     - Scheduler considers requests for placement

182. **Create Pod with resource limits.**
     - Limit: CPU 500m, Memory 512Mi
     - Container cannot exceed limits

183. **Create Pod where limit is lower than request.**
     - Request: 256Mi, Limit: 128Mi
     - Observe error or behavior
     - Understand why this is invalid

184. **Deploy Pod with Guaranteed QoS class.**
     - Request = Limit for CPU and Memory
     - Deploy pod
     - Pod gets Guaranteed QoS class

185. **Deploy Pod with Burstable QoS class.**
     - Requests < Limits
     - Pod gets Burstable QoS class

186. **Deploy Pod with BestEffort QoS class.**
     - No requests or limits
     - Pod gets BestEffort QoS class

---

## H. Init Containers and Lifecycle Hooks

187. **Create Pod with init container.**
     - Init container downloads config file
     - Main container uses config
     - Verify init runs first

188. **Create Pod with multiple init containers.**
     - Init1: download config
     - Init2: wait for service
     - Main: run application

189. **Create Pod with failed init container.**
     - Init container exits with error
     - Main container doesn't start
     - Observe pod status

190. **Use init container for security context setup.**
     - Init container runs as root
     - Setup permissions
     - Main container runs as non-root

191. **Create Pod with postStart hook.**
     - Hook executes after container starts
     - Hook downloads additional files
     - Verify files available in container

192. **Create Pod with preStop hook.**
     - Hook executes before container termination
     - Hook gracefully shuts down service
     - Allow time for hook to complete

193. **Test preStop hook timing.**
     - Set terminationGracePeriodSeconds: 30
     - Delete pod
     - Observe preStop hook has time to complete

---

# SERVICING AND NETWORKING (20%)

## A. Services

### ClusterIP Service

194. **Create ClusterIP Service for Deployment.**
     - Service exposes Deployment pods
     - Service accessible within cluster only
     - Verify service discovery via DNS

195. **Create Service with multiple ports.**
     - http: 80 -> 8080
     - https: 443 -> 8443
     - Name each port
     - Verify both ports work

196. **Create Service with named port in Deployment.**
     - Define port: http, containerPort: 8080
     - Reference named port in Service
     - Verify service uses correct port

197. **Create Service with specific ClusterIP.**
     - Assign ClusterIP: 10.96.1.100
     - Verify service has this IP

### NodePort Service

198. **Create NodePort Service.**
     - Expose Deployment on node port
     - Access from outside cluster
     - Verify port accessible on all nodes

199. **Create NodePort Service with specific port.**
     - nodePort: 30080
     - Verify accessible on that port

200. **Access NodePort service from different nodes.**
     - SSH to node1 and node2
     - Curl service IP:port
     - Verify service accessible from both nodes

### LoadBalancer Service

201. **Create LoadBalancer Service (cloud environment).**
     - Service type: LoadBalancer
     - External IP automatically assigned
     - Verify external IP accessible

202. **Create LoadBalancer Service with health check.**
     - Configure health check path
     - Verify load balancer checks service health

### Service Endpoint Discovery

203. **Understand Service endpoints.**
     - Create Service
     - View endpoints
     - Delete pod and observe endpoints updated

204. **Create Service with external endpoints.**
     - externalIPs: [192.168.1.1]
     - Service accessible via external IP

205. **Create Service with ExternalName.**
     - type: ExternalName
     - externalName: example.com
     - Service acts as alias to external service

### Session Affinity

206. **Create Service with session affinity.**
     - sessionAffinity: ClientIP
     - Multiple requests from same client go to same pod
     - Create test to verify

207. **Create Service with session affinity timeout.**
     - sessionAffinityConfig.clientIP.timeoutSeconds: 600
     - Session affinity expires after 10 minutes

---

## B. Endpoints and Endpoint Slices

208. **Create Endpoint manually.**
     - Create Service without selector
     - Create Endpoint manually pointing to external service
     - Deploy pod that uses Service

209. **Understand Endpoint Slices.**
     - View EndpointSlices
     - Understand advantages over Endpoints
     - Observe automatic creation

210. **Update Endpoint Slice.**
     - Add/remove addresses
     - Verify service traffic updates

---

## C. DNS and Service Discovery

### CoreDNS

211. **Verify CoreDNS pod running.**
     - Check coredns pod in kube-system namespace
     - View CoreDNS configuration

212. **Access service via DNS name.**
     - Deploy pod and service
     - Create test pod
     - Use nslookup to verify DNS resolution

213. **Understand DNS naming convention.**
     - Service name in same namespace: `service-name`
     - Service in different namespace: `service-name.namespace`
     - Fully qualified: `service-name.namespace.svc.cluster.local`

214. **Customize CoreDNS configuration.**
     - Edit CoreDNS ConfigMap
     - Add upstream DNS server
     - Verify configuration applied

215. **Configure CoreDNS for external DNS resolution.**
     - Setup external DNS plugin
     - Automatically create DNS records for services

### Service Discovery

216. **Environment variable-based service discovery.**
     - Deploy pod in same namespace as service
     - Environment variables set for service discovery
     - Access service via env vars

217. **DNS-based service discovery.**
     - Deploy pod in same namespace
     - Use DNS name to access service
     - Compare vs environment variables

218. **Headless Service and DNS.**
     - Create headless Service (clusterIP: None)
     - Deploy StatefulSet
     - Access pods via DNS names

---

## D. Ingress and Ingress Controllers

### Ingress Basics

219. **Create basic Ingress.**
     - Route domain.com to service
     - Deploy nginx ingress controller
     - Access via domain name

220. **Create Ingress with multiple paths.**
     - /api -> api-service
     - /web -> web-service
     - Verify routing works

221. **Create Ingress with TLS.**
     - Create TLS secret with cert and key
     - Ingress terminates HTTPS
     - Access via HTTPS

222. **Create Ingress with multiple hosts.**
     - api.domain.com -> api-service
     - web.domain.com -> web-service
     - Verify host-based routing

223. **Update Ingress rules.**
     - Modify path routing
     - Add new backend service
     - Verify changes applied

### Ingress Controllers

224. **Install nginx ingress controller.**
     - Deploy via Helm chart
     - Verify controller pods running
     - Verify LoadBalancer service created

225. **Install alternative ingress controller (e.g., Traefik).**
     - Deploy Traefik
     - Create Ingress for Traefik
     - Verify routing works

226. **Configure ingress controller with custom settings.**
     - Modify controller deployment
     - Add custom flags/settings
     - Verify changes applied

227. **Use ingress controller annotations.**
     - nginx.ingress.kubernetes.io/rewrite-target
     - Enable CORS
     - Add authentication
     - Verify annotations applied

### Ingress Policies

228. **Create Ingress with authentication.**
     - basic-auth annotation
     - Create secret with credentials
     - Access Ingress requires auth

229. **Create Ingress with rate limiting.**
     - Rate limit requests per IP
     - Verify rate limiting applied

230. **Create Ingress with CORS enabled.**
     - Enable cross-origin requests
     - Test from different origin

---

## E. Network Policies

### Basic Network Policies

231. **Create deny-all NetworkPolicy.**
     - Deny all incoming traffic to pods
     - Pods cannot receive traffic
     - Pods can send traffic

232. **Create NetworkPolicy allowing specific traffic.**
     - Allow traffic from label: tier=frontend
     - Block all other traffic
     - Deploy pods and test

233. **Create NetworkPolicy with port specification.**
     - Allow traffic on port 8080 only
     - Block traffic on other ports
     - Test with telnet/curl

234. **Create egress NetworkPolicy.**
     - Deny all outgoing traffic from pods
     - Allow only to specific services
     - Pods cannot reach external services

235. **Create NetworkPolicy for multi-namespace setup.**
     - Allow traffic from pods in namespace `production`
     - Deny traffic from other namespaces

### Advanced Network Policies

236. **Create NetworkPolicy with CIDR selector.**
     - Allow traffic from specific CIDR range
     - Test from within and outside range

237. **Create NetworkPolicy with protocol specification.**
     - Allow TCP port 443
     - Allow UDP port 53
     - Block other protocols

238. **Combine multiple NetworkPolicies.**
     - Multiple policies apply to same pods
     - Policies are OR'd together
     - All policies must allow for traffic to pass

239. **Create NetworkPolicy for Kubernetes-to-external communication.**
     - Allow pods to reach external API
     - Specify external IP in egress rule

240. **Test NetworkPolicy with debugging.**
     - Create test pods in different namespaces
     - Use tcpdump to verify traffic blocked
     - Verify Network Policy applied

---

## F. Gateway API (Advanced)

241. **Install Gateway API CRDs.**
     - Deploy Gateway API CustomResourceDefinitions
     - Verify installation

242. **Create Gateway resource.**
     - Define GatewayClass
     - Create Gateway
     - Configure listeners

243. **Create HTTPRoute for Gateway.**
     - Define HTTPRoute
     - Route to backend services
     - Test routing

244. **Compare Gateway API with Ingress.**
     - Understand differences
     - Advantages of Gateway API
     - When to use each

245. **Create advanced Gateway API routing.**
     - Path-based routing
     - Host-based routing
     - Weight-based traffic splitting

---

# STORAGE (10%)

## A. Persistent Volumes (PV) and Persistent Volume Claims (PVC)

### Basic PV and PVC

246. **Create Persistent Volume (PV).**
     - Capacity: 5Gi
     - Access mode: ReadWriteOnce
     - Storage class: manual
     - Path: /data/pv1

247. **Create Persistent Volume Claim (PVC).**
     - Request 2Gi storage
     - Access mode: ReadWriteOnce
     - Bind to PV created above

248. **Deploy Pod using PVC.**
     - Mount PVC in pod
     - Write file to mounted volume
     - Verify file persists

249. **Delete and recreate Pod.**
     - Delete pod
     - Create new pod with same PVC
     - Verify file still exists (data persisted)

250. **Understand PV lifecycle.**
     - Create PV
     - Create PVC binding to PV
     - Delete PVC
     - Check PV status (should be Released, not Available)

### PV Reclaim Policies

251. **Create PV with Retain reclaim policy.**
     - Delete PVC
     - PV should move to Released state
     - Data preserved, requires manual cleanup

252. **Create PV with Delete reclaim policy.**
     - Delete PVC
     - PV and underlying storage automatically deleted

253. **Create PV with Recycle reclaim policy (deprecated).**
     - Understand deprecation
     - Use Retain or Delete instead

### Access Modes

254. **Create PV with ReadWriteOnce access mode.**
     - PVC mounts to single pod
     - Try to mount to second pod
     - Second pod cannot mount

255. **Create PV with ReadWriteMany access mode.**
     - PVC mounts to multiple pods simultaneously
     - All pods read and write to same volume
     - Requires supported storage backend (NFS)

256. **Create PV with ReadOnlyMany access mode.**
     - Multiple pods mount volume in read-only mode
     - Pods cannot write

---

## B. Storage Classes and Dynamic Provisioning

### Basic Storage Classes

257. **Create Storage Class for dynamic provisioning.**
     - Provisioner: kubernetes.io/host-path
     - reclaimPolicy: Delete
     - allowVolumeExpansion: true

258. **Create PVC using Storage Class.**
     - PVC automatically provisions PV
     - Pod mounts PVC
     - Verify storage provisioned

259. **Deploy multiple PVCs from same Storage Class.**
     - Multiple PVCs provision multiple PVs
     - Each PV independent

260. **Set default Storage Class.**
     - Mark Storage Class as default
     - PVC without storageClassName uses default
     - Verify default used

### Storage Class Parameters

261. **Create Storage Class with parameters.**
     - type: fast (SSD)
     - replication-factor: 3
     - Provisioner uses parameters

262. **Create multiple Storage Classes with different parameters.**
     - bronze: regular storage
     - silver: SSD storage
     - gold: ultra-fast storage
     - Select appropriate class for workload

### Volume Expansion

263. **Expand PVC size.**
     - Increase requested storage
     - Pod detects and mounts expanded volume
     - Filesystem expands automatically

264. **Prevent volume expansion.**
     - allowVolumeExpansion: false in Storage Class
     - Try to expand PVC
     - Expansion should fail

---

## C. Volume Types

### ConfigMap and Secret Volumes

265. **Mount ConfigMap as volume.**
     - ConfigMap contains configuration files
     - Mount in pod
     - Files appear in specified directory

266. **Mount Secret as volume.**
     - Secret contains certificates and keys
     - Mount in pod
     - Verify permissions (usually 0600)

### emptyDir Volume

267. **Create Pod with emptyDir volume.**
     - Multiple containers share emptyDir
     - Data lost when pod deleted
     - Useful for temporary data between containers

268. **Use emptyDir with sizeLimit.**
     - Limit emptyDir to 1Gi
     - Fill beyond limit
     - Verify pod evicted

### hostPath Volume

269. **Create Pod with hostPath volume.**
     - Mount /var/log from node
     - Access node logs from container
     - Understand security implications

270. **Create Pod with hostPath type checking.**
     - type: Directory
     - Pod fails if directory doesn't exist

### downwardAPI Volume

271. **Create Pod with downwardAPI volume.**
     - Mount pod metadata as files
     - Access pod name, namespace, IP, etc.
     - Verify files contain correct data

### Projected Volume

272. **Create Pod with projected volume.**
     - Combine ConfigMap, Secret, downwardAPI
     - Single mount point for multiple sources
     - Verify all sources accessible

### NFS Volume

273. **Create NFS server.**
     - Setup NFS server with export
     - Configure permissions

274. **Mount NFS volume in Pod.**
     - NFS server: 192.168.1.100:/data
     - Mount path: /data
     - Verify shared storage

### iSCSI Volume

275. **Configure iSCSI initiator.**
     - Setup iSCSI target and initiator

276. **Mount iSCSI volume in Pod.**
     - Configure Pod with iSCSI volume
     - Mount iSCSI LUN
     - Verify mounted

---

## D. StatefulSets with Persistent Storage

277. **Create StatefulSet with volumeClaimTemplates.**
     - Each pod gets unique PVC
     - Storage persists with pod identity
     - Delete and recreate pod
     - Pod gets same storage

278. **Scale StatefulSet with persistent storage.**
     - Scale up: new PVCs created
     - Scale down: PVCs retained (for potential reattach)
     - Verify volumes preserved

279. **Update StatefulSet storage class.**
     - Create StatefulSet with old storage class
     - Change storage class
     - Understand limitations

---

## E. Snapshot and Restore

280. **Create VolumeSnapshot.**
     - Create snapshot of PVC
     - Use CSI driver supporting snapshots

281. **Restore PVC from snapshot.**
     - Create new PVC from snapshot
     - Restore data to new volume

282. **Create recurring snapshots.**
     - Setup schedule for automatic snapshots
     - Verify snapshots created
     - Test restore from old snapshot

---

# TROUBLESHOOTING (30%)

## A. Cluster and Node Troubleshooting

### Cluster Status

283. **Check cluster health.**
     - kubectl cluster-info
     - kubectl get componentstatuses
     - Verify all components healthy

284. **Verify API server is running.**
     - kubectl get pods -n kube-system
     - Find kube-apiserver pod
     - Check pod logs for errors

285. **Debug scheduler issues.**
     - kubectl logs kube-scheduler -n kube-system
     - Identify why pods not scheduled
     - Check scheduler configuration

286. **Debug controller manager issues.**
     - kubectl logs kube-controller-manager -n kube-system
     - Identify controller failures
     - Check replication controller status

### Node Troubleshooting

287. **Check node status.**
     - kubectl get nodes
     - Identify NotReady nodes
     - kubectl describe node to understand reason

288. **Check node capacity and allocation.**
     - kubectl describe nodes
     - Compare Capacity vs Allocatable vs Used
     - Identify resource constraints

289. **Drain node for maintenance.**
     - kubectl drain node-1 --ignore-daemonsets
     - Pods migrate to other nodes
     - Perform maintenance
     - kubectl uncordon node-1

290. **Debug kubelet on node.**
     - SSH to node
     - Check kubelet service: systemctl status kubelet
     - Check kubelet logs: journalctl -u kubelet -n 50

291. **Verify node network connectivity.**
     - SSH to node
     - Test DNS: nslookup kubernetes.default
     - Test API server connectivity: curl https://api-server:6443

292. **Check node disk pressure.**
     - kubectl describe node
     - Look for DiskPressure condition
     - Clean up unused images/containers

293. **Fix NotReady node.**
     - Investigate cause (CPU, Memory, Disk, Network)
     - Fix underlying issue
     - Node transitions to Ready

294. **Taint node to prevent scheduling.**
     - kubectl taint nodes node-1 key=value:NoSchedule
     - Debug reason
     - Remove taint after fix

---

## B. Pod Troubleshooting

### Pod Status

295. **Investigate Pending pod.**
     - kubectl describe pod
     - Check events for scheduling reasons
     - Common causes: insufficient resources, node not ready, affinity not satisfiable

296. **Investigate Failed pod.**
     - kubectl describe pod
     - Check container exit code
     - kubectl logs to see error output

297. **Investigate CrashLoopBackOff pod.**
     - Application crashing on startup
     - kubectl logs shows crash reason
     - Fix application and redeploy

298. **Investigate ImagePullBackOff pod.**
     - Image cannot be pulled from registry
     - Verify image name correct
     - Verify registry credentials
     - Check image exists in registry

299. **Investigate pod in Unknown state.**
     - Node communication lost
     - Kubelet on node not responding
     - Fix node connectivity

### Pod Logs and Debugging

300. **Get pod logs.**
     - kubectl logs pod-name
     - View application logs
     - Identify errors

301. **Get pod logs from previous container.**
     - Pod crashed and restarted
     - kubectl logs pod-name --previous
     - View logs from previous run

302. **Stream pod logs.**
     - kubectl logs -f pod-name
     - Follow logs in real-time
     - Useful for debugging

303. **Get logs from specific container in multi-container pod.**
     - kubectl logs pod-name -c container-name
     - Specify which container logs to view

304. **Debug pod with exec.**
     - kubectl exec -it pod-name -- /bin/bash
     - Start bash shell in container
     - Investigate container state manually

305. **Debug pod with port-forward.**
     - kubectl port-forward pod-name 8080:8080
     - Access pod service locally
     - Useful for debugging connectivity

306. **Debug pod with copy to local machine.**
     - kubectl cp pod-name:/path/to/file ./local-file
     - Copy files from pod for inspection

307. **Debug pod with describe.**
     - kubectl describe pod pod-name
     - View events and status
     - Understand scheduling decisions

308. **Debug pod with yaml output.**
     - kubectl get pod pod-name -o yaml
     - View full pod configuration
     - Understand how pod was deployed

### Pod Restart and Lifecycle

309. **Pod not starting after deployment.**
     - kubectl describe pod
     - Check events section
     - Common: wrong image, bad volume mount, resource limits

310. **Pod keeps restarting.**
     - CrashLoopBackOff state
     - Application error on startup
     - kubectl logs shows error

311. **Pod restart policy misbehaving.**
     - restartPolicy: Always but not restarting
     - restartPolicy: Never but restarting
     - Check Job specification

312. **Debug init container failure.**
     - Init container crashes
     - Main container doesn't start
     - kubectl logs shows init error
     - kubectl logs -c init-container-name

313. **Debug readiness probe failure.**
     - Pod keeps restarting due to readiness failure
     - Fix readiness probe logic
     - Common: endpoint not ready initially

314. **Debug liveness probe failure.**
     - Pod gets killed due to liveness probe failure
     - Application deadlocked or hung
     - Increase probe timeout

---

## C. Service and Networking Troubleshooting

### Service Debugging

315. **Service has no endpoints.**
     - kubectl get endpoints service-name
     - No pods matching service selector
     - Fix pod labels to match selector

316. **Pods selected by service but traffic not working.**
     - Verify endpoints created
     - Verify service selector labels match pod labels
     - Check port numbers match

317. **Service DNS not resolving.**
     - kubectl exec -it pod-name -- nslookup service-name
     - CoreDNS pod not running
     - CoreDNS configuration error
     - Fix CoreDNS pod or configuration

318. **Cannot access service from pod.**
     - kubectl exec -it pod-name -- curl service-name:port
     - Network Policy blocking traffic
     - Service port misconfigured
     - Pod not in service selector

319. **NodePort service not accessible.**
     - curl node-ip:node-port
     - Service not created properly
     - Node port out of range (30000-32767)
     - Firewall blocking port

320. **LoadBalancer service stuck in Pending.**
     - No external IP assigned
     - Cloud provider integration issue
     - Check provider configuration

### Network Policy Debugging

321. **Traffic blocked by Network Policy.**
     - kubectl get networkpolicies
     - kubectl describe networkpolicy
     - Verify policy selector matches pod
     - Check allowed source/destination

322. **Network Policy not working as expected.**
     - All pods can communicate despite policy
     - CNI plugin not supporting NetworkPolicy
     - Policy selector not matching pods
     - Fix policy or CNI configuration

323. **Egress Network Policy blocking external requests.**
     - Pod cannot reach external service
     - kubectl describe networkpolicy shows egress rules
     - Add external IP to egress whitelist

### Pod-to-Pod Communication

324. **Pods on different nodes cannot communicate.**
     - Check network plugin (CNI) running
     - Check pod network CIDR configured
     - Test with ping between pods
     - Check firewall rules

325. **Pods on same node cannot communicate.**
     - Likely Network Policy blocking
     - Or service misconfigured
     - Verify iptables rules: iptables -L -n

---

## D. Persistent Storage Troubleshooting

### PV and PVC Issues

326. **PVC stuck in Pending state.**
     - No PV available for binding
     - PV size/access mode doesn't match PVC
     - Create matching PV or use dynamic provisioning

327. **Pod cannot mount PVC.**
     - PVC not bound
     - Multiple pods trying to mount ReadWriteOnce PVC
     - Storage not accessible on node

328. **PV not binding to PVC.**
     - Storage class mismatch
     - Access mode incompatible
     - PV already bound to different PVC

329. **PVC cannot expand.**
     - Storage class doesn't allow expansion
     - Underlying storage doesn't support expansion
     - Enable allowVolumeExpansion in storage class

330. **Volume data lost after pod deletion.**
     - PV reclaim policy is Delete
     - Data disappeared when PVC deleted
     - Change to Retain for future

### Dynamic Provisioning Issues

331. **PVC not provisioning dynamically.**
     - Storage class not found
     - Provisioner not running
     - Insufficient resources for provisioning

332. **Persistent volume provisioner slow.**
     - Check provisioner logs
     - Check storage backend capacity
     - Monitor provisioner for errors

### Volume Mount Issues

333. **Pod cannot mount volume.**
     - Volume not accessible from node
     - Permission denied on volume
     - NFS mount permissions

334. **File permissions issue in mounted volume.**
     - Container cannot write to volume
     - Volume mounted read-only
     - Permission mismatch between node and container

335. **NFS volume mount timeout.**
     - NFS server not reachable
     - Firewall blocking NFS
     - NFS export not configured
     - Fix network connectivity

---

## E. Application and Container Troubleshooting

### Container Image Issues

336. **Image pull fails with authentication error.**
     - Private registry requires credentials
     - Create docker registry secret
     - Add imagePullSecrets to pod

337. **Image digest mismatch.**
     - Image expected hash doesn't match
     - Corrupted image
     - Re-pull image

338. **Container image too large.**
     - Image pull timeout on slow network
     - Optimize image size
     - Use image pull policy: IfNotPresent

### Application Errors

339. **Application in pod crashes with segmentation fault.**
     - Application code bug
     - Get pod logs and stack trace
     - Fix application

340. **Application runs out of memory.**
     - Pod OOMKilled
     - Increase memory limit
     - Fix memory leak in application

341. **Application port conflicts.**
     - Multiple pods trying to bind same port
     - Use different ports for each pod
     - Or separate into different deployments

342. **Application cannot reach database.**
     - Database service DNS not resolving
     - Database pod not running
     - Network Policy blocking access
     - Fix application configuration

343. **Application slow performance.**
     - High latency to database
     - Insufficient CPU/Memory
     - Increase resource limits
     - Optimize application

---

## F. Resource and Quota Troubleshooting

### Resource Requests/Limits

344. **Pod evicted due to memory pressure.**
     - Multiple pods competing for memory
     - Node runs out of memory
     - Increase node memory or limit pod replicas
     - Set memory requests/limits appropriately

345. **Pod unable to schedule due to insufficient resources.**
     - Requested resources exceed available on any node
     - Add more nodes
     - Reduce requested resources
     - Reduce pod replicas

346. **CPU throttling occurring.**
     - Pod CPU limit too low
     - Increase CPU limit
     - Monitor actual CPU usage

### ResourceQuota

347. **Pod creation fails with quota exceeded.**
     - Namespace ResourceQuota limit reached
     - kubectl describe resourcequota
     - Delete some resources or increase quota

348. **Understand ResourceQuota impact.**
     - Pod requests count towards quota
     - Storage requests count towards quota
     - Number of pods counts towards quota limit

### Namespace LimitRange

349. **Pod rejected by LimitRange.**
     - Pod requests below minimum
     - Pod limits above maximum
     - Adjust pod resources to match LimitRange

350. **No default limits on pod.**
     - Create default LimitRange
     - New pods get default limits
     - Existing pods unaffected

---

## G. Control Plane Component Troubleshooting

### API Server Debugging

351. **API server not responding.**
     - kubectl commands hang or error
     - Check API server pod: kubectl get pods -n kube-system
     - Check API server logs: kubectl logs -n kube-system kube-apiserver-node-name
     - Restart API server if needed

352. **API server authentication failures.**
     - Check certificates: openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
     - Check kubeconfig
     - Verify RBAC policies

353. **API server request latency high.**
     - Slow etcd queries
     - High API server load
     - Monitor API server metrics

### Scheduler Debugging

354. **Scheduler not scheduling pods.**
     - kubectl logs -n kube-system kube-scheduler
     - Check for errors in scheduling
     - Verify scheduler running

355. **Pods scheduled to wrong nodes.**
     - Check scheduling algorithm
     - Review node affinity rules
     - Check node labels

### Controller Manager Debugging

356. **Deployments not rolling out.**
     - Controller manager issue
     - Check controller manager logs
     - Verify service account permissions

357. **Replicas not matching desired.**
     - ReplicaSet controller not running
     - Check controller manager pod
     - Verify RBAC for controller

### etcd Debugging

358. **etcd cluster unhealthy.**
     - kubectl get pods -n kube-system
     - Check etcd pod logs
     - Check member list: etcdctl member list
     - Fix cluster member issues

359. **etcd too slow.**
     - High latency on etcd operations
     - Check network latency between etcd members
     - Check disk IO on etcd storage
     - Optimize etcd performance

360. **etcd corruption.**
     - Restore from backup
     - Rebuild etcd cluster
     - Verify data integrity

---

## H. Multi-Cluster and Advanced Troubleshooting

### Cluster Federation

361. **Pod not synced to federated clusters.**
     - Federation controller not running
     - Check federation controller logs
     - Verify cluster registered in federation

### Custom Resources

362. **Custom resource controller not running.**
     - Check operator pod logs
     - Verify CRD exists
     - Check webhook configuration

363. **Custom resource stuck in pending.**
     - Operator not processing CR
     - Check operator logs for errors
     - Verify operator permissions (RBAC)

### Webhook Debugging

364. **Mutating webhook not working.**
     - kubectl get mutatingwebhookconfigurations
     - Check webhook service is running
     - Test webhook manually

365. **Validating webhook rejecting valid requests.**
     - Check validation rules in webhook
     - Review webhook logs
     - Test request format

### Advanced Debugging Techniques

366. **Capture API traffic with proxy.**
     - kubectl proxy
     - curl localhost:8001/api/v1/...
     - Monitor all API traffic

367. **Debug with strace on kubelet.**
     - SSH to node
     - strace -p $(pgrep kubelet) -o trace.log
     - Identify system calls causing issues

368. **Monitor cluster with Prometheus.**
     - Deploy Prometheus
     - Configure Kubernetes service monitor
     - Query metrics for troubleshooting

369. **Check audit logs.**
     - Review API audit logs
     - Identify suspicious API calls
     - Trace user actions

370. **Use debug containers.**
     - kubectl debug pod-name
     - Start debug container in pod namespace
     - Investigate pod environment

---

## I. Common Troubleshooting Scenarios (Real Exam Like)

### Scenario 1: Pod Not Scheduling

371. **SCENARIO: Pod in Pending state, not scheduling.**
     - Requirements:
       - Diagnose reason (affinity, resources, node selector, etc.)
       - Fix issue
       - Verify pod now runs
     - Steps:
       - kubectl describe pod
       - Check events
       - Identify exact reason
       - Fix underlying cause

### Scenario 2: Service Not Accessible

372. **SCENARIO: Service created but pods cannot access it.**
     - Requirements:
       - Create service, deployment, test pods
       - Verify service endpoints
       - Verify DNS resolution
       - Verify Network Policy allows access
       - Debug until service accessible
     - Document steps to fix

### Scenario 3: Node Not Ready

373. **SCENARIO: Node showing NotReady status.**
     - Requirements:
       - SSH to node
       - Check kubelet service
       - Check kubelet logs
       - Verify network connectivity
       - Fix issue
       - Verify node becomes Ready
     - Show all debugging steps

### Scenario 4: High Memory Usage

374. **SCENARIO: Nodes showing memory pressure, pods being evicted.**
     - Requirements:
       - Identify pods with highest memory usage
       - Increase memory limits or reduce replicas
       - Add node or optimize application
       - Verify evictions stop
       - Monitor memory usage

### Scenario 5: PVC Not Binding

375. **SCENARIO: PVC stuck in Pending, PV not binding.**
     - Requirements:
       - Check PV exists
       - Verify access modes compatible
       - Verify storage classes match (if using)
       - Debug binding issue
       - Fix and verify binding

### Scenario 6: Ingress Not Working

376. **SCENARIO: Created Ingress but cannot access via hostname.**
     - Requirements:
       - Verify ingress controller running
       - Check ingress object created correctly
       - Verify service and pods exist
       - Test local DNS resolution
       - Verify TLS cert if HTTPS
       - Debug until accessible

### Scenario 7: Pod Cannot Pull Private Image

377. **SCENARIO: Pod ImagePullBackOff, cannot get private image.**
     - Requirements:
       - Create docker-registry secret
       - Add to pod imagePullSecrets
       - Verify image pulled and pod runs
     - Document complete solution

### Scenario 8: Storage Write Permission Denied

378. **SCENARIO: Pod with PVC volume mount, cannot write to volume.**
     - Requirements:
       - Verify pod can read from volume
       - Diagnose write permission issue
       - Fix permissions
       - Verify pod can write
     - Consider SELinux, volume permissions, etc.

### Scenario 9: StatefulSet Pod Stuck

379. **SCENARIO: StatefulSet pod stuck in Pending.**
     - Requirements:
       - Investigate why pod cannot schedule
       - Check PVC status
       - Check affinity rules
       - Fix blocking issue
       - Verify pod schedules

### Scenario 10: Multiple Services on Same Node

380. **SCENARIO: Port conflict when multiple services expose same port.**
     - Requirements:
       - Understand why conflict occurs
       - Use different node ports
       - Or use different services
       - Resolve conflict
       - Verify both services accessible

---

## J. Additional Advanced Troubleshooting Scenarios

### Scenario 11: Control Plane Upgrade Failure

381. **SCENARIO: Cluster upgrade interrupted, etcd corrupted.**
     - Requirements:
       - Diagnose corruption
       - Restore from backup
       - Complete upgrade
       - Verify all components operational

### Scenario 12: Network Segmentation

382. **SCENARIO: Network Policy overly restrictive, some traffic blocked.**
     - Requirements:
       - Identify blocked traffic
       - Update NetworkPolicy rules
       - Verify all necessary traffic allowed
       - Maintain security posture

### Scenario 13: RBAC Permission Denied

383. **SCENARIO: User cannot perform necessary operations, RBAC denied.**
     - Requirements:
       - Verify user identity
       - Check RBAC roles and bindings
       - Add necessary permissions
       - Verify user can now perform operations

### Scenario 14: Pod Termination Issues

384. **SCENARIO: Pod taking too long to terminate, stuck in Terminating state.**
     - Requirements:
       - Check preStop hook
       - Check termination grace period
       - Force delete if necessary
       - Identify cause and prevent recurrence

### Scenario 15: Kubelet Communication Issues

385. **SCENARIO: Kubelet not communicating with API server.**
     - Requirements:
       - SSH to node
       - Check kubelet logs
       - Verify API server certificate validity
       - Fix connectivity
       - Node transitions to Ready

---

# Additional Advanced Topics

## Advanced Scheduling Scenarios

386. **Create pod with node affinity preferring specific zone but allowing fallback.**
     - Primary: zone=us-west-1a
     - Fallback: any zone
     - Weights for preference levels

387. **Create pod with pod affinity preferring same rack as database but allowing other racks.**
     - Similar to question 386 but for pod affinity

388. **Implement topology spreading for high availability.**
     - Use topologySpreadConstraints
     - Spread pods across zones and nodes
     - Ensure even distribution

389. **Create pod that must run on GPU node, cannot fall back.**
     - Required node affinity for gpu=true
     - Pod stays pending if no GPU nodes
     - Use in ML workload scenario

390. **Schedule batch job on nodes with disk=ssd only.**
     - Fast I/O requirement
     - Node selector or node affinity
     - Verify only on SSD nodes

---

## Advanced Networking Scenarios

391. **Create Ingress with hostname-based TLS routing.**
     - Multiple TLS certs for multiple hosts
     - Host-based routing with different backend services

392. **Create service mesh scenario with network policies.**
     - Multiple services communicating through mesh
     - Network policies enforcing service-to-service rules
     - Only specific services can communicate

393. **Configure split brain scenario with multiple ingress controllers.**
     - Two ingress controllers on same cluster
     - Route different domains to different controllers
     - Load balance across controllers

---

## Advanced Storage Scenarios

394. **Create persistent volume with complex backup/restore scenario.**
     - Snapshot volume before data deletion
     - Restore from snapshot
     - Verify data recovered completely

395. **Multi-replica stateful application with persistent storage.**
     - Master-slave setup
     - Each instance has persistent storage
     - Verify replication works
     - Test failover

---

## Advanced RBAC Scenarios

396. **Create complex RBAC with multiple teams and roles.**
     - Team A: read-only on production
     - Team B: full access on development
     - Team C: specific service management access

397. **Implement audit trail with RBAC.**
     - Track which user made what changes
     - Review audit logs
     - Identify unauthorized access attempts

398. **Create role for multi-tenant isolation.**
     - Each tenant namespace has own roles
     - Prevent cross-tenant access
     - Verify isolation works

---

## Advanced Troubleshooting Scenarios

399. **Diagnose race condition causing pod scheduling issues.**
     - Two deployments competing for resources
     - Pods stuck in pending
     - Analyze scheduler logs
     - Fix resource allocation

400. **Debug complex network policy interaction.**
     - Multiple network policies on same pod
     - Understand how they combine
     - Fix unintended blocking

---

# Tips for CKA Exam Preparation

## General Tips

- **Time Management**: Each question worth ~2 minutes. Practice solving quickly.
- **Kubectl Mastery**: Know common commands by heart
  - `kubectl get`, `kubectl describe`, `kubectl logs`
  - `kubectl edit`, `kubectl apply`, `kubectl delete`
  - `kubectl exec`, `kubectl port-forward`, `kubectl port-forward`
- **Documentation**: You can access Kubernetes official documentation during exam
  - Bookmark important pages for quick access
  - Practice finding solutions in documentation
- **Dry Run**: Use `--dry-run=client -o yaml` to generate manifests quickly
- **Copy/Paste**: Practice efficient copying from documentation examples

## Command Reference

```bash
# Quick Pod Creation
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

# Quick Deployment Creation
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml

# Quick ConfigMap Creation
kubectl create configmap my-config --from-literal=key=value --dry-run=client -o yaml

# Quick Secret Creation
kubectl create secret generic my-secret --from-literal=password=secret --dry-run=client -o yaml

# Scale Deployment
kubectl scale deployment nginx --replicas=5

# Update Image
kubectl set image deployment/nginx nginx=nginx:1.20

# Port Forward
kubectl port-forward pod/nginx 8080:80

# Execute Command in Pod
kubectl exec -it pod-name -- /bin/bash

# View Logs
kubectl logs pod-name

# Describe Resource
kubectl describe pod pod-name

# Get YAML of Resource
kubectl get pod pod-name -o yaml

# Apply Manifest
kubectl apply -f manifest.yaml

# Delete Resource
kubectl delete pod pod-name

# Label Node
kubectl label nodes node-1 disk=ssd

# Taint Node
kubectl taint nodes node-1 gpu=true:NoSchedule

# Drain Node
kubectl drain node-1 --ignore-daemonsets

# Uncordon Node
kubectl uncordon node-1
```

## Study Strategy

1. **Understand Concepts**: Don't just memorize, understand the "why"
2. **Hands-On Practice**: Create manifests from scratch, not just copy-paste
3. **Troubleshooting Focus**: 30% of exam is troubleshooting, practice debugging
4. **Lab Environment**: Setup local Kubernetes cluster for practice (minikube, kind)
5. **Timed Practice**: Solve practice questions under time pressure
6. **Review Failures**: When you fail a question, understand why and learn
7. **Official Documentation**: Familiarize yourself with Kubernetes documentation
8. **Real-World Scenarios**: Practice applying knowledge to real scenarios

---

# End of CKA Question Bank

**Total Questions**: 400+ comprehensive practical questions
**Coverage**: All CKA exam domains
**Level**: Professional exam-level difficulty
**Format**: Real-world practical scenarios

Good luck with your CKA exam preparation!

