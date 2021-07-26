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
ibmcloud resource groups

ibmcloud target -r us-south -g <default-group-id>

ibmcloud ce project list

# -k for the kube config
ibmcloud ce project create -n ray-test

ibmcloud ce project select -n ray-test -k

export NAMESPACE=$(kubectl get namespace --no-headers | awk '{ print $1 }')

curl -L https://gist.github.com/ruediger-maass/042ef187aa012f77d2a83b46ecda4df8/raw/8e8ce900829e96739cca059c05488642c63d6aa0/example-cluster.yaml | sed "s/NAMESPACE/$NAMESPACE/" > example-cluster.yaml

# cat example-cluster.yaml
```

## Ray
```
ray up example-cluster.yaml
```


## Issues

1) Code Engine seems to only accept request/limits in specific CPU and Mem combinations. To remedy, edit `example-cluster.yaml` and modify each request/limit mem value to `2G` (which is the smallest value that seems to be accepted based on the error message)

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
