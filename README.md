# cnat

The `cnat` (cloud native at) command extends Kubernetes to run a command at a certain point in time in the future, akin to the Linux [at](https://en.wikipedia.org/wiki/At_(command)) command.

Let's say you want to execute `echo YAY` at 2am on 3rd July 2019. Here's what you would do (given the `cnat` CRD has been registered and the according operator is running):

```bash
$ cat runat.yaml
apiVersion: cnat.programming-kubernetes.info/v1alpha1
kind: At
metadata:
  name: example-at
spec:
  schedule: "2019-07-03T02:00:00Z"
  command: "echo YAY"


$ kubectl apply -f runat.yaml
cnat.programming-kubernetes.info/example-at created

$ kubectl get at
NAME               AGE
example-at         20s

$ kubectl logs example-at-pod
YAY
```

Note that the execution time (`schedule`) is given in [UTC](https://www.utctime.net/).

We follow the [sample-controller](https://github.com/kubernetes/sample-controller)
along with the [client-go library](https://github.com/kubernetes/client-go).

It makes use of the generators in [k8s.io/code-generator](https://github.com/kubernetes/code-generator) to generate a typed client, informers, listers and deep-copy functions. You can do this yourself using the `./hack/update-codegen.sh` script.

The `update-codegen` script will automatically generate the following:

* `pkg/apis/samplecontroller/v1alpha1/zz_generated.deepcopy.go`
* `pkg/generated/`

Changes should not be made to these files manually, and when creating your own
controller based off of this implementation you should not copy these files and
instead run the `update-codegen` script to generate your own.

```bash
# build custom controller binary:
$ go build -o cnat-controller .

# launch custom controller locally:
$ ./cnat-controller -kubeconfig=$HOME/.kube/config

# after API/CRD changes:
$ ./hack/update-codegen.sh

# register At custom resource definition:
$ kubectl apply -f artifacts/examples/cnat-crd.yaml

# create an At custom resource:
$ kubectl apply -f artifacts/examples/cnat-example.yaml
```
