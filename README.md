# Ray on IBM Cloud Code Engine

## Prereq

```
ibmcloud plugin install code-engine

pip install -U https://s3-us-west-2.amazonaws.com/ray-wheels/latest/ray-2.0.0.dev0-cp37-cp37m-macosx_10_13_intel.whl
pip install kubernetes
pip install 'ray[default]'
```

## Target RG and Create Code Engine Project
```
# List resource groups
ibmcloud resource groups

# Target US South and Default resource group
ibmcloud target -r us-south -g Default

# See your existing Code Engine projects
ibmcloud ce project list

# Create project if needed
ibmcloud ce project create -n ray-test

# -k for setting your local kubeconfig
ibmcloud ce project select -n ray-test -k

# Save to variable for reuse
export NAMESPACE=$(kubectl get namespace --no-headers | awk '{ print $1 }')

# Download example cluster YAML
curl -L https://gist.github.com/ruediger-maass/042ef187aa012f77d2a83b46ecda4df8/raw/8e8ce900829e96739cca059c05488642c63d6aa0/example-cluster.yaml | sed "s/NAMESPACE/$NAMESPACE/" > example-cluster.yaml

# Check content
# cat example-cluster.yaml
```

## Ray
```
ray up example-cluster.yaml
```
Some issues are observed after running `ray up` using the provided `example-cluster.yaml` file. Please see the "Issues" section for details and workarounds.

```
# Check logs
ray exec ./example-cluster.yaml 'tail -n 100 -f /tmp/ray/session_latest/logs/monitor*'
```


```
# Port Forward the pod to access dashboard
ray dashboard example-cluster.yaml &

# Then navigate to 
http://localhost:8265
```


## Issues

1) Code Engine seems to only accept request/limits in specific CPU and Mem combinations. To remedy, edit `example-cluster.yaml` and modify each request/limit mem value to `500m` and `1G` respectively. Then run `ray up example-cluster.yaml` again


```
{
    "kind": "Status",
    "status": "Failure",
    "details": {
        "name": "example-cluster-ray-head-cdvcs",
        "causes": [
            {
                "reason": "ResourceRequestNotAllowed",
                "message": "Cpu/Mem requested: 1 / 1Gi for container ray-node - Requested Cpu/Mem not in list of allowed values: [{CPU:125m Mem:250M} {CPU:250m Mem:500M} {CPU:500m Mem:1G} {CPU:1 Mem:2G} {CPU:2 Mem:4G} {CPU:4 Mem:8G} {CPU:6 Mem:12G} {CPU:8 Mem:16G} {CPU:125m Mem:500M} {CPU:250m Mem:1G} {CPU:500m Mem:2G} {CPU:1 Mem:4G} {CPU:2 Mem:8G} {CPU:4 Mem:16G} {CPU:6 Mem:24G} {CPU:8 Mem:32G} {CPU:125m Mem:1G} {CPU:250m Mem:2G} {CPU:500m Mem:4G} {CPU:1 Mem:8G} {CPU:2 Mem:16G} {CPU:4 Mem:32G}]",
                "field": "/spec/template/spec/containers/i/resources"
            }
        ],
        "kind": "Pod"
    },
    "metadata": {},
    "message": "admission webhook \"validating.webhook.serving.kube.codeengine.cloud.ibm.com\" denied the request: Resource Configuration error:\nCpu/Mem requested: 1 / 1Gi for container ray-node - Requested Cpu/Mem not in list of allowed values: [{CPU:125m Mem:250M} {CPU:250m Mem:500M} {CPU:500m Mem:1G} {CPU:1 Mem:2G} {CPU:2 Mem:4G} {CPU:4 Mem:8G} {CPU:6 Mem:12G} {CPU:8 Mem:16G} {CPU:125m Mem:500M} {CPU:250m Mem:1G} {CPU:500m Mem:2G} {CPU:1 Mem:4G} {CPU:2 Mem:8G} {CPU:4 Mem:16G} {CPU:6 Mem:24G} {CPU:8 Mem:32G} {CPU:125m Mem:1G} {CPU:250m Mem:2G} {CPU:500m Mem:4G} {CPU:1 Mem:8G} {CPU:2 Mem:16G} {CPU:4 Mem:32G}]",
    "reason": "Status Unprocessable Entity",
    "apiVersion": "v1",
    "code": 422
}
```

2) **Update: No longer seeing the following error on July 28th.**  After running `ray up` another error occurs. 

```
kubernetes.client.exceptions.ApiException: (422)
Reason: Unprocessable Entity
HTTP response headers: HTTPHeaderDict({'Date': 'Mon, 26 Jul 2021 20:07:43 GMT', 'Content-Type': 'application/json', 'Content-Length': '1804', 'Connection': 'keep-alive', 'Audit-Id': '0e063020-c8b7-4e3f-a831-f91dd1770e89', 'Cache-Control': 'no-cache, private', 'X-Kubernetes-Pf-Flowschema-Uid': '5bc69b35-ecf3-452f-9444-ac4464cb3580', 'X-Kubernetes-Pf-Prioritylevel-Uid': 'd1839302-c341-41b1-a216-264fac31c3ca', 'Strict-Transport-Security': 'max-age=31536000; includeSubDomains', 'X-Content-Type-Options': 'nosniff', 'Content-Security-Policy': "default-src 'none'; frame-ancestors 'none'", 'X-XSS-Protection': '1; mode=block'})
HTTP response body: {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"Pod \"example-cluster-ray-head-lqxvp\" is invalid: spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds` or `spec.tolerations` (only additions to existing tolerations)\n  core.PodSpec{\n  \t... // 10 identical fields\n  \tAutomountServiceAccountToken: nil,\n  \tNodeName:                     \"10.240.64.124\",\n  \tSecurityContext: \u0026core.PodSecurityContext{\n  \t\t... // 11 identical fields\n  \t\tFSGroupChangePolicy: nil,\n  \t\tSysctls:             nil,\n- \t\tSeccompProfile:      nil,\n+ \t\tSeccompProfile:      \u0026core.SeccompProfile{Type: \"RuntimeDefault\"},\n  \t},\n  \tImagePullSecrets: nil,\n  \tHostname:         \"\",\n  \t... // 15 identical fields\n  }\n","reason":"Invalid","details":{"name":"example-cluster-ray-head-lqxvp","kind":"Pod","causes":[{"reason":"FieldValueForbidden","message":"Forbidden: pod updates may not change fields other than `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds` or `spec.tolerations` (only additions to existing tolerations)\n  core.PodSpec{\n  \t... // 10 identical fields\n  \tAutomountServiceAccountToken: nil,\n  \tNodeName:                     \"10.240.64.124\",\n  \tSecurityContext: \u0026core.PodSecurityContext{\n  \t\t... // 11 identical fields\n  \t\tFSGroupChangePolicy: nil,\n  \t\tSysctls:             nil,\n- \t\tSeccompProfile:      nil,\n+ \t\tSeccompProfile:      \u0026core.SeccompProfile{Type: \"RuntimeDefault\"},\n  \t},\n  \tImagePullSecrets: nil,\n  \tHostname:         \"\",\n  \t... // 15 identical fields\n  }\n","field":"spec"}]},"code":422}
```

