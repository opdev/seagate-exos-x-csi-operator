# Seagate Exos X CSI Operator

The Seagate Exos X CSI Operator offers a convenient way to deploy and manage
the
[Seagate Exos X CSI driver](https://github.com/Seagate/seagate-exos-x-csi).
It was built with [operator-sdk](https://sdk.operatorframework.io).

## Deployment

There are three ways to run/install the Operator:

* Create a
  [Operator Lifecycle Manager CatalogSource](https://olm.operatorframework.io/docs/concepts/crds/catalogsource/)
  and install the Operator via the OpenShift Console/UI
* Run the `operator-sdk run bundle` command
* Run the Operator-SDK's `make run` target outside the OpenShift cluster.

### OLM CatalogSource and install via the OpenShift UI

To install the Operator through the Openshift UI, we first create a index:

```sh
$ opm index add --bundles quay.io/opdev/seagate-exos-x-csi-operator-bundle:v0.0.6 --tag quay.io/opdev/seagate-exos-x-csi-operator-index:0.0.6 -c docker
```

Once the index is created, we will push the index image to any repository

```sh
$ docker push quay.io/opdev/seagate-exos-x-csi-operator-index:0.0.6
```

Next, we will be creating a CatalogSource and adding the newly created
operator index image to the catalogSource. The CatalogSource file is present
in this repository

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: my-test-operators
  namespace: openshift-marketplace
spec:
  sourceType: grpc
  image: quay.io/opdev/seagate-exos-x-csi-operator-index:0.0.6
  displayName: Test Operators
  publisher: Red Hat Partner
```

```sh
$ oc create -f catalogsource.yaml
```

Once, the CatalogSource is created, we will go to the Openshift UI and install
the Operator through the OperatorHub.

![OperatorHub Tile](https://user-images.githubusercontent.com/1616123/140323013-05e5a50b-0017-42b8-b0be-e4537850a9d8.png)

![Install Operator](https://user-images.githubusercontent.com/1616123/140323026-93d5dd44-57bf-47eb-af6e-698f57cd8027.png)

The Operator will be depoyed in the `openshift-operators` namespace.

### Operator-SDK Run Bundle

Bundle your operator, then build and push the bundle image. The bundle target
generates a bundle in the bundle directory containing manifests and metadata
defining your Operator. bundle-build and bundle-push build and push a bundle
image defined by bundle.Dockerfile.

```sh
$ make bundle bundle-build bundle-push
```

Make sure that the bundle image is public.

```sh
$ operator-sdk run bundle quay.io/opdev/seagate-exos-x-csi-operator-bundle:v0.0.6
```

### Run outside the OpenShift cluster(`make run`)

To run the operator as a go program outside the cluster we will use the
following command:

```sh
$ make run
```

## Creating the CR

Once the controller/operator is running, we want to create our custom CR in
the `config/sample` directory:

```sh
$ oc apply -f config/samples/storage_v1alpha1_seagateexosxcsi.yaml
```

To verify the creation of the `SeagateExosXCsi` CR:

```sh
$ oc get seagateexosxcsis
NAME                              AGE
seagateexosxcsi-sample            1m
```

Verify the creation of the Operand:

```sh
$ oc --kubeconfig=.kube/config get allNAME                            READY   STATUS    RESTARTS      AGE
pod/seagate-exos-x-csi-controller-server-697b5d9785-68nz4             5/5     Running   0             103s
pod/seagate-exos-x-csi-node-server-frpnl                              3/3     Running   0             102s
pod/seagate-exos-x-csi-node-server-rvvvz                              3/3     Running   0             103s
pod/seagate-exos-x-csi-node-server-t5dp5                              3/3     Running   0             102s
pod/seagate-exos-x-csi-operator-controller-manager-667b457fcc-t8gvw   2/2     Running   1 (94s ago)   3m19s

NAME                                                                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/seagate-exos-x-csi-operator-controller-manager-metrics-service   ClusterIP   172.30.246.233   <none>        8443/TCP   3m21s

NAME                                            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/seagate-exos-x-csi-node-server   3         3         3       3            3           <none>          104s

NAME                                                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/seagate-exos-x-csi-controller-server             1/1     1            1           104s
deployment.apps/seagate-exos-x-csi-operator-controller-manager   1/1     1            1           3m20s

NAME                                                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/seagate-exos-x-csi-controller-server-697b5d9785             1         1         1       104s
replicaset.apps/seagate-exos-x-csi-operator-controller-manager-667b457fcc   1         1         1       3m20s
```

Alternatively, the Custom Resource can be created from the UI as well.

![Create CR](https://user-images.githubusercontent.com/1616123/139892149-292d567a-445f-49f9-94f8-6680e3b0a652.png)

## Cleanup

Clean up the SeagateExosXCSI CR first:

```sh
$ oc delete -f config/samples/storage_v1alpha1_seagateexosxcsi.yaml
```

**Note:** Make sure the above custom resource has been deleted before
proceeding to stop the Operator program. Otherwise your cluster may have
dangling custom resource objects that cannot be deleted.
