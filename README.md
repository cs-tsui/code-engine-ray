# Ray on IBM Cloud Code Engine

## Prereq

```
ibmcloud plugin install code-engine

pip install -U https://s3-us-west-2.amazonaws.com/ray-wheels/latest/ray-2.0.0.dev0-cp37-cp37m-macosx_10_13_intel.whl
pip install kubernetes
pip install 'ray[default]'
```

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
