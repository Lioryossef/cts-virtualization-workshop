# Exercise 1 - OpenShift Virtualization Installation

Navigate to the top-level `Operators` menu entry, and select `OperatorHub`. This lists all of the available operators that you can install from the operator catalogue. Start typing 'virtualization' in the search box and you should see an entry called "OpenShift Virtualization".

<img width="1395" alt="Screen Shot 2022-07-19 at 2 30 03" src="https://user-images.githubusercontent.com/64369864/179633707-ab6d6ee5-afec-4044-befd-9837d2103582.png">

Next you'll want to select the 'Install' button. Leave the defaults here as they'll automatically select the latest version of OpenShift Virtualization, and will be placed into a new "openshift-cnv" project.

<img width="993" alt="Screen Shot 2022-07-19 at 2 31 31" src="https://user-images.githubusercontent.com/64369864/179633859-628660f1-75f8-4dc9-af66-15f4bc6bdd87.png">

Next we need to deploy the **HyperConverged** resource, which, in addition to the OpenShift Virtualization operator, creates and maintains an OpenShift Virtualization Deployment for the cluster.

<img width="501" alt="Screen Shot 2022-07-19 at 2 34 07" src="https://user-images.githubusercontent.com/64369864/179634092-59547c13-f5b6-4db7-875c-4cac939c9dd9.png">

> This may raise some questions about a similar term: "hyperconverged infrastructures" and the relationship of workloads and storage associated with that term. For OpenShift Virtualization this relates to the fact that we're converging virtual machines and containers and is how an "instance" of OpenShift Virtualization is instantiated - it does not impact the relation between compute and storage as we will see later in the labs. 

Click on "**Create HyperConverged**", as a required operand, in the same screen to proceed.

This will open a new screen. We can again accept all the defaults for this lab - for real world use, there are many additional flags, parameters, and attributes that can be applied at this stage, such as enabling tech-preview features, specifying host devices, implementing CPU masks, and so on.

Continue the installation by clicking on "**Create**" at the bottom.

<img width="996" alt="Screen Shot 2022-07-17 at 16 43 38" src="https://user-images.githubusercontent.com/64369864/179401246-ba142016-d678-4755-b944-10aa48fc3bb5.png">

You can move to the `Workloads` --> `Pods` menu entry and watch it start all of its resources

<img width="870" alt="Screen Shot 2022-07-19 at 2 37 34" src="https://user-images.githubusercontent.com/64369864/179634433-0845bd20-cdd1-47e8-9c9d-1c11b2b955e8.png">

Watch via the CLI

```
watch -n1 'oc get pods -n openshift-cnv'
```

You will know the process is complete when you see that the operator installation has been successful by running the following command

```
oc get csv -n openshift-cnv
```

You should see the output below

```
NAME                                       DISPLAY                    VERSION   REPLACES                                   PHASE
kubevirt-hyperconverged-operator.v4.10.2   OpenShift Virtualization   4.10.2    kubevirt-hyperconverged-operator.v4.10.1   Succeeded
```

If you do not see `Succeeded` in the `PHASE` column then the deployment may still be progressing, or has failed. You will not be able to proceed until the installation has been successful. Once the `PHASE` changes to `Succeeded` you can validate that the required resources and the additional components have been deployed across the nodes. First let's check the pods deployed in the `openshift-cnv` namespace

```execute-1
oc get pods -n openshift-cnv
```

This will list the list of pods in *openshift-cnv* project.

~~~bash
NAME                                                  READY   STATUS    RESTARTS   AGE
bridge-marker-52mv7                                   1/1     Running   0          6m55s
bridge-marker-74rzv                                   1/1     Running   0          6m55s
bridge-marker-8h7gd                                   1/1     Running   0          6m55s
bridge-marker-hvdx2                                   1/1     Running   0          6m55s
bridge-marker-lsx48                                   1/1     Running   0          6m55s
bridge-marker-sbmhr                                   1/1     Running   0          6m55s
cdi-apiserver-5b4cfb4d57-4zd72                        1/1     Running   0          6m55s
cdi-deployment-65d77bbf54-bwvtk                       1/1     Running   0          6m55s
cdi-operator-cfbb47d55-5nxhk                          1/1     Running   0          13m
cdi-uploadproxy-5dddc647cb-m8mmp                      1/1     Running   0          6m53s
cluster-network-addons-operator-679485cff4-qkjm9      1/1     Running   0          13m
hco-operator-667cdd75d7-7hmml                         1/1     Running   0          13m
hco-webhook-5c8d75b559-d4h5r                          1/1     Running   0          13m
(...)
~~~

> **NOTE**: All pods shown from this command should be in the `Running` state. You will have many different types, the above snippet is just an example of the output at one point in time, you may have more or less at any one point. Below we discuss some of the pod types and what they do.

You may check by using grep to do inverse-filtering for any 'Running' pods

```execute-1
oc get pods -n openshift-cnv | grep -v Running
```

This should return an empty result, i.e. ALL pods are running successfully

~~~bash
NAME                                                  READY   STATUS    RESTARTS   AGE
~~~


Together, all of these pods are responsible for various functions of running a virtual machine on-top of OpenShift/Kubernetes. See the table below that describes some of the various different pod types and their function

| Pod Name                             | Pod Responsibilities |
| ------------------------------------ | -------------------- |
| *[bridge-marker](https://github.com/kubevirt/bridge-marker)*                      | Marks network bridges as available node resources.|
| *[cdi-*](https://github.com/kubevirt/containerized-data-importer)*                              | The Containerized Data Importer (CDI) is a Kubernetes extension to populate PVCs with VM disk images or other data. CDI pods allow OpenShift Virtualization to import, upload and clone Virtual Machine images. |
| *[cluster-network-addons-operator](https://github.com/kubevirt/cluster-network-addons-operator)*    | Allows the installation of additional networking plugins. |
| *[hco-operator](https://github.com/kubevirt/hyperconverged-cluster-operator)*                       | Allows users to deploy and configure multiple operators in a single operator and via a single entry point. An "operator of operators." |
| *[hostpath-provisioner-operator](https://github.com/kubevirt/hostpath-provisioner-operator)*      |Operator that manages the hostpath-provisioner, which provisions storage on network filesystems mounted on the host.|
| *[kube-cni-linux-bridge-plugin](https://github.com/containernetworking/plugins)*       |CNI Plugin to create a network bridge and add a host and container to it.|
| *kubemacpool-mac-controller-manager* |Allocation of MAC addresses from a pool to secondary interfaces.|
| *[kubevirt-node-labeller](https://github.com/kubevirt/node-labeller)*             |Creates node labels based on CPU (and other hardware) information.|
| *[kubevirt-ssp-operator](https://github.com/MarSik/kubevirt-ssp-operator)*              |Scheduling, Scale and Performance operator for OpenShift. The Hyperconverged Cluster Operator automatically installs the SSP operator when deploying.|
| *nmstate-handler*                    |Deploys NMState which allows network administrators to manage host networking settings in a declarative manner.|
| *[node-maintenance-operator](https://github.com/kubevirt/cluster-network-addons-operator#nmstate)*|Operator that allows the administrator to deploy the NMState State Controller.                    |
| *[virt-api](https://github.com/kubevirt/kubevirt/tree/master/pkg/virt-api)*                           |Kubernetes Virtualization API and runtime in order to define and manage virtual machines|
| *[virt-controller](https://kubernetes.io/blog/2018/05/22/getting-to-know-kubevirt/)*                    |The operator that’s responsible for cluster-wide virtualisation functionality|
| *[virt-handler](https://kubernetes.io/blog/2018/05/22/getting-to-know-kubevirt/)*                       |Tracks changes to a VM's state.|
| *[virt-template-validator](https://kubernetes.io/blog/2018/05/22/getting-to-know-kubevirt/)*            |Add-on to check the annotations on templates and reject them if invalid.|


There's also a few custom resources that get defined too. For example the `NodeNetworkState` (`nns`)  provides the current network configuration of our nodes - this is used to verify whether physical networking configurations have been successfully applied by the `nmstate-handler` pods. This is useful for ensuring that the NetworkManager state on each node is configured as required. We use this for defining interfaces/bridges on each of the machines for both physical machine connectivity and for providing network access for pods (and virtual machines) within OpenShift/Kubernetes. View the NodeNetworkState state with

```execute-1
oc get nns -A
```

Which shows the list of nodes being managed

~~~bash
NAME                                                     AGE
workshop-n54ln-master-0.c.almog-elfassy.internal         6m27s
workshop-n54ln-master-1.c.almog-elfassy.internal         6m11s
workshop-n54ln-master-2.c.almog-elfassy.internal         6m44s
workshop-n54ln-worker-b-cqvfp.c.almog-elfassy.internal   6m48s
workshop-n54ln-worker-c-x98pv.c.almog-elfassy.internal   6m49s
workshop-n54ln-worker-d-rdcx9.c.almog-elfassy.internal   6m57s
~~~

You can then view the details of each managed node with

```execute-1
oc get nns/workshop-n54ln-worker-b-cqvfp.c.almog-elfassy.internal -o yaml
```

This shows the NodeNetworkState definition in *YAML* format
~~~yaml
apiVersion: nmstate.io/v1beta1
kind: NodeNetworkState
metadata:
  name: workshop-n54ln-worker-b-cqvfp.c.almog-elfassy.internal
(...)
    interfaces:
    - ipv4:
        address: []
        enabled: false
      ipv6:
        address: []
        enabled: false
      mac-address: 8A:4D:D5:52:27:40
      mtu: 1410
      name: br0
      state: down
      type: ovs-interface
    - ipv4:
        address:
        - ip: 10.0.0.8
          prefix-length: 32
        auto-dns: true
        auto-gateway: true
        auto-route-table-id: 0
        auto-routes: true
        dhcp: true
        enabled: true
      ipv6:
        address:
        - ip: fe80::aac1:727:4258:3f30
          prefix-length: 64
        auto-dns: true
        auto-gateway: true
        auto-route-table-id: 0
        auto-routes: true
        autoconf: true
        dhcp: true
        enabled: true
      lldp:
        enabled: false
      mac-address: 42:01:0A:00:00:08
      mtu: 1460
      name: ens4
      state: up
      type: ethernet
(...)
~~~

```execute-1
oc get nnce -n openshift-cnv
```

It should not find any *nnce* definitions yet, as we've not defined any:
~~~bash
No resources found
~~~

# Viewing the OpenShift Virtualization Dashboard
When OpenShift Virtualization is deployed it adds additional components to OpenShift's web console so you can interact with objects and custom resources defined by OpenShift Virtualization, including VirtualMachine types. You can now navigate to "**Virtualization**" on the left-hand side panel, and you should see the new snap-in component for OpenShift Virtualization.

<img width="1095" alt="Screen Shot 2022-07-19 at 2 39 15" src="https://user-images.githubusercontent.com/64369864/179634572-a1b6f411-62c3-4767-b571-03a5fba616c3.png">

>  **Please don't try and create any virtual machines just yet, we'll get to that shortly!**

