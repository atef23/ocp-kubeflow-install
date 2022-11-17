# ocp-kubeflow-install

Log in to openshift

1. 
```
git clone https://github.com/opendatahub-io/manifests.git --branch v1.6-branch-openshift
cd manifests/
```

2. 
```
oc create ns kubeflow
```

3. Create Kubeflow user namespace (where kubeflow will run applications and notebooks)
```
oc create ns kubeflow-user-example-com
```

4. Apply manifests
```
while ! kustomize build openshift/example/istio | kubectl apply -f -; do echo "Retrying to apply resources"; sleep 10; done
```

4. Monitor pod status in kubeflow namespace
```
oc get pods -n kubeflow
```

5. Once the pods are up, get the kubeflow UI route 
```
oc get routes -n istio-system istio-ingressgateway -o jsonpath='http://{.spec.host}/'
```

5. Once the pods are up, get the kubeflow UI route 
```
oc get routes -n istio-system istio-ingressgateway -o jsonpath='http://{.spec.host}/'
```

5. Access the UI from the route above. When prompted, create user with name "kf-workspace"

6. Apply the following yaml in kf-workspace to allow access to Kubeflow pipelines
```
apiVersion: kubeflow.org/v1alpha1
kind: PodDefault
metadata:
  name: access-ml-pipeline
  namespace: kf-workspace
spec:
  desc: Allow access to Kubeflow Pipelines
  selector:
    matchLabels:
      access-ml-pipeline: "true"
  volumes:
    - name: volume-kf-pipeline-token
      projected:
        sources:
          - serviceAccountToken:
              path: token
              expirationSeconds: 7200
              audience: pipelines.kubeflow.org      
  volumeMounts:
    - mountPath: /var/run/secrets/kubeflow/pipelines
      name: volume-kf-pipeline-token
      readOnly: true
  env:
    - name: KF_PIPELINES_SA_TOKEN_PATH
      value: /var/run/secrets/kubeflow/pipelines/token
```

7. In a text editor, replace all instances of "kubeflow-user-example-com" with "kf-workspace" (must be done after logging in because of kubeflow UI bug, UI doesnt let you log in as kubeflow-user-example-com)

8. Reapply resources to allow security permissions on service accounts in kf-workspace namespace:
```
while ! kustomize build openshift/example/istio | kubectl apply -f -; do echo "Retrying to apply resources"; sleep 10; done
```
