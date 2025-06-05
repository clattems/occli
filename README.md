# OpenShift CLI (oc) Cheat Sheet

## Installation and Setup

### Installing Bash Completion
```bash
# Install bash completion for oc (Linux)
oc completion bash | sudo tee /etc/bash_completion.d/oc > /dev/null

# Install for current user only (Linux)
oc completion bash > ~/.oc_completion
echo 'source ~/.oc_completion' >> ~/.bashrc

# Install on macOS with Homebrew
brew install bash-completion
oc completion bash > $(brew --prefix)/etc/bash_completion.d/oc

# Install on macOS manually
oc completion bash > ~/.oc_completion
echo 'source ~/.oc_completion' >> ~/.bash_profile

# For zsh users
oc completion zsh > ~/.oc_completion_zsh
echo 'source ~/.oc_completion_zsh' >> ~/.zshrc

# Verify completion is working (restart shell first)
oc get <TAB><TAB>
oc describe pod <TAB><TAB>
```

### Shell Setup
```bash
# Add oc to your PATH if not already there
export PATH=$PATH:/path/to/oc

# Set default editor for oc edit commands
export EDITOR=vim

# Enable kubectl compatibility mode
alias kubectl=oc
```

## Getting Started

### Login and Authentication
```bash
# Log into OpenShift cluster
oc login https://api.cluster-url.com:6443

# Login with specific user
oc login -u username -p password https://api.cluster-url.com:6443

# Check current user
oc whoami

# Get login token for scripting
oc whoami -t
```

## Context Management

### Working with Projects (Namespaces)
```bash
# List all projects you can access
oc projects

# Switch to a different project
oc project my-project-name

# Get current project
oc project

# Create new project
oc new-project my-new-project --description="My project description"

# Delete project
oc delete project my-project-name
```

### Cluster Context
```bash
# Show current cluster context
oc config current-context

# List all available contexts
oc config get-contexts

# Switch between contexts
oc config use-context my-context-name

# View full config
oc config view
```

## Getting Information

### Events and Troubleshooting
```bash
# Get events in current project
oc get events --sort-by='.lastTimestamp'

# Get events for specific namespace
oc get events -n my-namespace --sort-by='.lastTimestamp'

# Get recent events (last hour)
oc get events --sort-by='.lastTimestamp' --field-selector involvedObject.kind=Pod

# Get events for specific pod
oc get events --field-selector involvedObject.name=my-pod-name

# Get warning events only
oc get events --field-selector type=Warning

# Get events with specific reason
oc get events --field-selector reason=Failed

# Watch events in real-time
oc get events -w

# Get events in JSON format for parsing
oc get events -o json

# Get formatted event output
oc get events -o wide
```

### Services and Routes
```bash
# List services
oc get services
oc get svc

# List routes (external access points)
oc get routes

# Get route URL
oc get route my-route -o jsonpath='{.spec.host}'

# List ingress controllers
oc get ingresscontroller -n openshift-ingress-operator
```

## Network Information

### Network Policies and Security
```bash
# List network policies
oc get networkpolicy

# Get cluster network configuration
oc get network.config.openshift.io cluster -o yaml

# List security context constraints
oc get scc

# Check which SCC a service account uses
oc get pod my-pod -o yaml | grep scc
```

### Cluster Network Information
```bash
# Get cluster network configuration
oc get clusternetwork

# Get detailed cluster network info with CIDR ranges
oc get clusternetwork default -o yaml

# List host subnets assigned to nodes
oc get hostsubnet

# Get specific node's subnet allocation
oc get hostsubnet node-name -o yaml

# Get network operator configuration
oc get network.operator cluster -o yaml

# Check cluster network status
oc get clusteroperator network -o yaml

# Get network type (OpenShiftSDN, OVNKubernetes, etc.)
oc get network.config.openshift.io cluster -o jsonpath='{.spec.networkType}'

# List all network-related cluster operators
oc get clusteroperator | grep -i network
```

### Network Diagnostics
```bash
# Get node network information
oc get nodes -o wide

# List cluster network operator status
oc get clusteroperator network

# Get cluster DNS information
oc get dns.operator cluster -o yaml

# Test network connectivity from a pod
oc exec my-pod -- curl -I http://service-name:port

# Get pod network details
oc get pods -o wide

# Check network plugin pods
oc get pods -n openshift-sdn
oc get pods -n openshift-ovn-kubernetes
```

### Load Balancer and Ingress
```bash
# Get ingress controllers
oc get ingresscontroller -n openshift-ingress-operator

# List load balancer services
oc get svc --field-selector spec.type=LoadBalancer

# Get external IPs
oc get svc -o wide
```

## Output Parsing with jq

### Basic JSON Output
```bash
# Get pod names only
oc get pods -o json | jq -r '.items[].metadata.name'

# Get pod status
oc get pods -o json | jq -r '.items[] | "\(.metadata.name): \(.status.phase)"'

# Get container images
oc get pods -o json | jq -r '.items[].spec.containers[].image'

# Get node resource usage
oc get nodes -o json | jq -r '.items[] | "\(.metadata.name): CPU=\(.status.capacity.cpu) Memory=\(.status.capacity.memory)"'
```

### Advanced jq Filtering
```bash
# Get only running pods
oc get pods -o json | jq -r '.items[] | select(.status.phase=="Running") | .metadata.name'

# Get pods with resource limits
oc get pods -o json | jq -r '.items[] | select(.spec.containers[].resources.limits) | .metadata.name'

# Get service endpoints
oc get endpoints -o json | jq -r '.items[] | "\(.metadata.name): \(.subsets[].addresses[].ip)"'

# Get persistent volume claims with storage size
oc get pvc -o json | jq -r '.items[] | "\(.metadata.name): \(.spec.resources.requests.storage)"'

# Parse events for specific namespace with jq
oc get events -n my-namespace -o json | jq -r '.items[] | "\(.lastTimestamp) \(.type) \(.reason): \(.message)"'

# Get cluster network CIDR information
oc get clusternetwork -o json | jq -r '.items[] | "Network: \(.network) CIDR: \(.clusterNetworks[].cidr) Host Prefix: \(.clusterNetworks[].hostSubnetLength)"'

# Get host subnet assignments
oc get hostsubnet -o json | jq -r '.items[] | "\(.host): \(.subnet)"'
```

### Network-Specific jq Queries
```bash
# Get all external routes with their hosts
oc get routes -o json | jq -r '.items[] | "\(.metadata.name): https://\(.spec.host)"'

# Get service ports
oc get svc -o json | jq -r '.items[] | "\(.metadata.name): \(.spec.ports[].port)"'

# Get ingress controller external IPs
oc get svc -n openshift-ingress -o json | jq -r '.items[] | select(.spec.type=="LoadBalancer") | "\(.metadata.name): \(.status.loadBalancer.ingress[].ip)"'
```

## Logs and Debugging

### Log Management
```bash
# Get pod logs
oc logs my-pod-name

# Follow logs in real-time
oc logs -f my-pod-name

# Get logs from previous container restart
oc logs my-pod-name --previous

# Get logs from specific container in multi-container pod
oc logs my-pod-name -c container-name

# Get recent logs with timestamp
oc logs my-pod-name --since=1h --timestamps
```

### Debugging Commands
```bash
# Execute command in pod
oc exec my-pod-name -- command

# Get shell access to pod
oc exec -it my-pod-name -- /bin/bash

# Port forward to local machine
oc port-forward my-pod-name 8080:8080

# Copy files to/from pod
oc cp my-file.txt my-pod-name:/tmp/
oc cp my-pod-name:/tmp/my-file.txt ./local-file.txt
```

### Creating Kubeconfig Files for Users
```bash
# Extract current kubeconfig cluster info
oc config view --minify --raw > my-cluster-config.yaml

# Create kubeconfig for service account
oc create serviceaccount my-service-account
oc create clusterrolebinding my-service-account-binding --clusterrole=view --serviceaccount=default:my-service-account

# Get service account token (OpenShift 4.11+)
TOKEN=$(oc create token my-service-account)

# Get cluster info for kubeconfig
CLUSTER_NAME=$(oc config view --minify -o jsonpath='{.clusters[0].name}')
CLUSTER_SERVER=$(oc config view --minify -o jsonpath='{.clusters[0].cluster.server}')
CLUSTER_CA=$(oc config view --minify --raw -o jsonpath='{.clusters[0].cluster.certificate-authority-data}')

# Create kubeconfig file for service account
cat > service-account-kubeconfig.yaml << EOF
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: ${CLUSTER_CA}
    server: ${CLUSTER_SERVER}
  name: ${CLUSTER_NAME}
contexts:
- context:
    cluster: ${CLUSTER_NAME}
    user: my-service-account
    namespace: default
  name: my-service-account-context
current-context: my-service-account-context
users:
- name: my-service-account
  user:
    token: ${TOKEN}
EOF

# Test the new kubeconfig
KUBECONFIG=service-account-kubeconfig.yaml oc get pods

# Create kubeconfig for regular user (certificate-based)
# First, create a certificate signing request
openssl genrsa -out myuser.key 2048
openssl req -new -key myuser.key -out myuser.csr -subj "/CN=myuser/O=mygroup"

# Create CSR in OpenShift
cat > csr.yaml << EOF
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myuser-csr
spec:
  request: $(cat myuser.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF

oc apply -f csr.yaml

# Approve the CSR (requires admin privileges)
oc certificate approve myuser-csr

# Get the signed certificate
oc get csr myuser-csr -o jsonpath='{.status.certificate}' | base64 -d > myuser.crt

# Create kubeconfig for certificate-based user
cat > user-kubeconfig.yaml << EOF
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: ${CLUSTER_CA}
    server: ${CLUSTER_SERVER}
  name: ${CLUSTER_NAME}
contexts:
- context:
    cluster: ${CLUSTER_NAME}
    user: myuser
    namespace: default
  name: myuser-context
current-context: myuser-context
users:
- name: myuser
  user:
    client-certificate-data: $(cat myuser.crt | base64 | tr -d '\n')
    client-key-data: $(cat myuser.key | base64 | tr -d '\n')
EOF
```

### Managing Multiple Kubeconfig Files
```bash
# Merge kubeconfig files
KUBECONFIG=~/.kube/config:./new-config.yaml oc config view --merge --flatten > merged-config.yaml

# Switch between different kubeconfig files
export KUBECONFIG=./service-account-kubeconfig.yaml
oc get pods

# Use specific kubeconfig for single command
KUBECONFIG=./user-kubeconfig.yaml oc get nodes

# Set default kubeconfig location
export KUBECONFIG=~/.kube/my-custom-config

# Copy context from one kubeconfig to another
oc config use-context source-context
oc config view --minify --raw > temp-context.yaml
KUBECONFIG=~/.kube/config:temp-context.yaml oc config view --merge --flatten > ~/.kube/config
```

### Token Management for Service Accounts
```bash
# Create long-lived token secret (OpenShift 4.10 and earlier method)
cat > sa-token-secret.yaml << EOF
apiVersion: v1
kind: Secret
metadata:
  name: my-service-account-token
  annotations:
    kubernetes.io/service-account.name: my-service-account
type: kubernetes.io/service-account-token
EOF

oc apply -f sa-token-secret.yaml

# Get token from secret
TOKEN=$(oc get secret my-service-account-token -o jsonpath='{.data.token}' | base64 -d)

# Create temporary token (expires after specified time)
oc create token my-service-account --duration=24h

# Create token for specific audience
oc create token my-service-account --audience=my-application
```

## RBAC and Security

### User Impersonation
```bash
# Impersonate another user
oc --as=username get pods

# Impersonate user with specific group
oc --as=username --as-group=system:authenticated get pods

# Impersonate service account
oc --as=system:serviceaccount:namespace:serviceaccount-name get pods

# Test what you can do as another user
oc --as=username auth can-i create pods

# Check permissions across all namespaces as another user
oc --as=username auth can-i create pods --all-namespaces
```

### Permission Analysis
```bash
# Check what a user can do (requires who-can plugin)
oc who-can create pods

# Check who can perform action in specific namespace
oc who-can create pods -n my-namespace

# Check who can perform action on specific resource
oc who-can get pod/my-pod

# List who can perform cluster-wide actions
oc who-can create clusterroles

# Check permissions for service accounts
oc who-can create pods --as=system:serviceaccount:default:my-sa

# Get all users who can perform any action on a resource type
oc who-can '*' pods
```

### Current User Permissions
```bash
# Check if current user can perform an action
oc auth can-i create pods

# Check permissions in specific namespace
oc auth can-i create pods -n my-namespace

# Check cluster-wide permissions
oc auth can-i create clusterroles

# List all permissions for current user
oc auth can-i --list

# Check permissions for specific resource
oc auth can-i get pod/my-pod

# Check with specific verb
oc auth can-i delete deployments.apps
```

## API Documentation and Resource Discovery

### Understanding Resources with oc explain
```bash
# Get basic information about a resource type
oc explain pod

# Get detailed field descriptions
oc explain pod.spec

# Explore nested fields
oc explain pod.spec.containers

# Get all available fields for a resource
oc explain pod --recursive

# Explain custom resources
oc explain route.spec

# Get API version information
oc explain deployment.apps/v1

# Explore specific field with examples
oc explain pod.spec.containers.resources

# Get information about cluster-scoped resources
oc explain clusterrole

# Explain network-related resources
oc explain networkpolicy.spec
oc explain service.spec.ports
```

### API Resource Discovery
```bash
# List all available API resources
oc api-resources

# Get API resources with their short names
oc api-resources --namespaced=true

# List cluster-scoped resources only
oc api-resources --namespaced=false

# Get API versions
oc api-versions

# Find resources by kind
oc api-resources | grep -i network

# Get detailed API resource information
oc api-resources -o wide
```

## Resource Management

### Creating and Applying Resources
```bash
# Apply configuration from file
oc apply -f my-config.yaml

# Create from template
oc process -f template.yaml | oc apply -f -

# Scale deployment
oc scale deployment my-app --replicas=3

# Delete resources
oc delete pod my-pod-name
oc delete -f my-config.yaml
```

### Resource Monitoring
```bash
# Get resource usage
oc top nodes
oc top pods

# Get detailed resource information with jq
oc get pods -o json | jq -r '.items[] | "\(.metadata.name): CPU=\(.spec.containers[].resources.requests.cpu // "none") Memory=\(.spec.containers[].resources.requests.memory // "none")"'
```

## Useful Aliases

Add these to your shell profile for faster access:

```bash
alias k='oc'
alias kgp='oc get pods'
alias kgs='oc get svc'
alias kgr='oc get routes'
alias kdp='oc describe pod'
alias kl='oc logs'
alias kx='oc exec -it'
```

## Quick Tips

### Working with Multiple Clusters
When managing multiple OpenShift clusters, always verify your current context before running commands:
```bash
oc config current-context && oc project
```

### JSON Path for Specific Data
Use jsonpath for extracting specific fields without jq:
```bash
# Get pod IPs
oc get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.podIP}{"\n"}{end}'

# Get node external IPs
oc get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.addresses[?(@.type=="ExternalIP")].address}{"\n"}{end}'
```

### Batch Operations
```bash
# Restart all pods in a deployment
oc rollout restart deployment/my-deployment

# Delete all pods with specific label
oc delete pods -l app=my-app

# Get all resources with specific label
oc get all -l environment=production
```
