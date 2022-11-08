Now we're going to clone a workload and see that it's identical to the source of the clone. 

- Create a Fedora 34 VM
- Create a NodePort service for ssh
- Launch it as a virtual machine via OpenShift Virtualization 
- Install a basic application inside the VM.
- Clone the VM
- Test the clone to make sure it's identical to the source

### Virtual Machine

Now let's create a VM based on the Fedora image, that will become our original VM that we'll clone in a later step. For review we are using the PVC with CDI importing the Fedora image (stored on ODF/OCS), and we are utilising the standard networking on the workers - the same as we've been using for the other virtual machines we created previously

 Make sure you're in your project

<img width="1058" alt="Screen Shot 2022-07-28 at 0 37 24" src="https://user-images.githubusercontent.com/64369864/181377172-0f761db6-a3a4-4544-8d75-d7c7871e31a8.png">

<img width="1351" alt="Screen Shot 2022-07-26 at 23 30 29" src="https://user-images.githubusercontent.com/64369864/181106241-85fc309d-5e9e-46d0-9309-eaaab50adb20.png">

Create the VM and expose SSH access with the existing public key

![image](https://user-images.githubusercontent.com/64369864/181780665-e35ca6db-81b5-45bc-a736-38e48073d559.png)

Copy the id_rsa.pub from the bastion and paste in the authorized Key wizard

For example - `ssh-ed25519 AAAAC3NzaC11ZDI1NTE5AAAAIA4EBVQf3WMiI0M3M740bOuEu/LARJLGH/7Jr1rlKCrylab-user@provision`

![image](https://user-images.githubusercontent.com/64369864/181974297-56416bd4-8e83-4c9b-ae1f-0c7eca8ba192.png)

CLI - Make sure you're in your project

```execute-1
oc project < workshop >
```

You should see 

~~~bash
Now using project "workshop" on server "https://api.workshop.aelfassy.com:6443".
~~~

We can view the running VM

```execute-1
oc get vmi
```

You should see the following, noting that your IP address may be different

~~~bash
NAME                 AGE     PHASE     IP             NODENAME                                                 READY
fedora-workshop      31m     Running   10.128.2.78    workshop-n54ln-worker-c-x98pv.c.almog-elfassy.internal   True
~~~

When you've got an IP address, we should be able to SSH to it by the NodePort service. A NodePort exposes the service on a static port on the node’s IP address. NodePorts are in the 30000 to 32767 range by default.

```execute-1
oc get svc
```
You should see the static port, in this example the port is 30736

~~~bash
NAME                          TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
fedora-workshop-ssh-service   NodePort   172.30.143.119   <none>        22000:30736/TCP   3h57m
~~~

SSH to our VM with the `private key`

```copy
ssh -i id_rsa fedora@api.almog-elfassy.internal -p 30736
```

The following tasks should be performed from the VM's shell

~~~bash
[fedora@fedora-workshop ~]$
~~~

We're going to deploy a basic application into our Fedora-based virtual machine; let's install `nginx` via `systemd` and `podman`, i.e. have *systemd* call *podman* to start an *nginx* container at boot time, and have it display a simple web page

First change to root user

```execute-1
sudo su
```

install podman

```execute-1
dnf install podman -y
```

Then create an nginx service based on podman

```execute-1
cat >> /etc/systemd/system/nginx.service << EOF
[Unit]
Description=Nginx Podman container
Wants=syslog.service
[Service]
ExecStart=/usr/bin/podman run --net=host quay.io/aelfassy/images/nginxvirt:ip
ExecStop=/usr/bin/podman stop --all
[Install]
WantedBy=multi-user.target
EOF
```

Then enable nginx service

```execute-1
systemctl daemon-reload && systemctl enable --now nginx
```

Which should show

~~~bash
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /etc/systemd/system/nginx.service.
~~~

Check nginx service status

```execute-1
systemctl status nginx
```

It should show the service as "active (running)":

~~~bash
● nginx.service - Nginx Podman container
   Loaded: loaded (/etc/systemd/system/nginx.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2020-03-31 01:30:48 UTC; 8s ago
 Main PID: 9898 (podman)
    Tasks: 11 (limit: 2345)
~~~

Now logout from the VM

```execute-1
logout
```

Let's create one more VM (**by YAML**) that will be as clinet for our application and use Centos image.

First, make sure you're in your project

```execute-1
oc project < workshop >
```

You should see 

~~~bash
Now using project "workshop" on server "https://api.workshop.aelfassy.com:6443".
~~~

Now create a Centos VirtualMachine using **YAML**

```execute-1
cat << EOF | oc apply -f -
apiVersion: kubevirt.io/v1alpha3
kind: VirtualMachine
metadata:
  name: centos-workshop
  labels:
    app: centos-workshop
    os.template.kubevirt.io/centos7.0: 'true'
    vm.kubevirt.io/template-namespace: openshift
    workload.template.kubevirt.io/server: 'true'
spec:
  dataVolumeTemplates:
    - apiVersion: cdi.kubevirt.io/v1beta1
      kind: DataVolume
      metadata:
        creationTimestamp: null
        name: centos
      spec:
        sourceRef:
          kind: DataSource
          name: centos7
          namespace: openshift-virtualization-os-images
        storage:
          accessModes:
            - ReadWriteMany
          resources:
            requests:
              storage: 30Gi
          storageClassName: ocs-storagecluster-ceph-rbd
          volumeMode: Block
  running: true
  template:
    metadata:
      labels:
        vm.kubevirt.io/name: centos-workshop
        flavor.template.kubevirt.io/small: 'true'
        kubevirt.io/size: small
    spec:
      domain:
        cpu:
          cores: 1
          sockets: 1
          threads: 1
        devices:
          disks:
            - bootOrder: 1
              disk:
                bus: virtio
              name: centos
            - disk:
                bus: virtio
              name: cloudinitdisk
          interfaces:
            - name: default
              masquerade: {}
          networkInterfaceMultiqueue: true
          rng: {}
        machine:
          type: pc-q35-rhel8.4.0
        resources:
          requests:
            memory: 2Gi
      evictionStrategy: LiveMigrate
      hostname: centos-workshop
      networks:
        - name: default
          pod: {}
      terminationGracePeriodSeconds: 0
      volumes:
        - dataVolume:
            name: centos
          name: centos
        - cloudInitNoCloud:
            userData: |-
              #cloud-config
              user: centos
              password: aelfassy
              chpasswd: { expire: False }
          name: cloudinitdisk
EOF
```

This should start a new VM

~~~bash
virtualmachine.kubevirt.io/centos-workshop created
~~~

After a few minutes, this VM should be started, and we can check with this command

```execute-1
oc get vmi
```

We should then see our VM running

~~~bash
NAME                             AGE   PHASE     IP            NODENAME                                                 READY
fedora-workshop                  25m   Running   10.128.2.78   workshop-n54ln-worker-d-x98pv.c.almog-elfassy.internal   True
centos-workshop                  68s   Running   10.129.2.75   workshop-n54ln-worker-c-x98pv.c.almog-elfassy.internal   True
~~~

We can see the Virtual Machine Instance is created on the *pod network*, note the IP address in the **10.12x** range. If you recall, all VMs are managed by pods, and the pod manages the networking. So we should see the same IP address on the pod associated with the VM:

```execute
oc get pods -o wide
```

Which clearly shows that the IP address the VM has matches the IP of the virt-launcher pod, noting that your IP addresses may be different to the example, but should match

~~~bash
NAME                                                 READY   STATUS    RESTARTS   AGE   IP            NODE              NOMINATED NODE   READINESS GATES
virt-launcher-fedora-workshop-j45c9                  1/1     Running   0          28m   10.128.2.78   workshop-n54ln-worker-d-x98pv.c.almog-elfassy.internal    <none>           1/1
virt-launcher-centos-workshop-5vcsc                  1/1     Running   0          26h   10.129.2.75   workshop-n54ln-worker-c-x98pv.c.almog-elfassy.internal   <none>           1/1
~~~

We can also check the *pod* for the networks-status, showing the same IP address. Remember to change the pod name to reflect your environment:

```copy
$ oc describe pod/virt-launcher-centos-workshop-5vcsc | grep -A 9 networks-status
```

~~~bash
              k8s.v1.cni.cncf.io/networks-status:
                [{
                    "name": "openshift-sdn",
                    "interface": "eth0",
                    "ips": [
                        "10.129.2.75"
                    ],
                    "default": true,
                    "dns": {}
                }]
~~~

One of the models available for OpenShift Virtualization is to provide networking with a combination of attachments, including "pod networking". This mean we can have virtual machines attached to the same networks that the container pods are attached to. This has the added benefit of allowing virtual machines to leverage all of the Kubernetes models for services, load balancers, ingress, network policies, node ports, and a wide variety of other functions.

Pod networking is also referred to as "masquerade mode" when it's related to OpenShift Virtualization, and it can be used to hide a virtual machine’s outgoing traffic behind the pod IP address. Masquerade mode uses Network Address Translation (NAT) to connect virtual machines to the Pod network backend through a Linux bridge. Masquerade mode is the recommended binding method for VM's that need to use (or have access to) the default pod network.

> As the VM is running within a pod itself and being hosted within the same cluster, you should be able to ping and get the app directly from our Fedora VM 

First access to our **VM client** (centos) by console

<img width="597" alt="Screen Shot 2022-07-29 at 15 02 50" src="https://user-images.githubusercontent.com/64369864/181754474-bdbc8c81-bdfe-44f2-83a4-ccd07d5ec5ce.png">

Check ping to our Fedora VM (app)

```copy
ping -c4 10.128.2.78
```

Which should return

~~~bash
$ ping -c4 10.128.2.78
PING 10.128.2.78 (10.128.2.78) 56(84) bytes of data.
64 bytes from 10.128.2.78: icmp_seq=1 ttl=63 time=1.69 ms
64 bytes from 10.128.2.78: icmp_seq=2 ttl=63 time=1.69 ms
64 bytes from 10.128.2.78: icmp_seq=3 ttl=63 time=1.69 ms
64 bytes from 10.128.2.78: icmp_seq=4 ttl=63 time=1.69 ms

--- 10.128.2.78 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.692/1.692/1.692/0.000 ms
~~~

Take a look around and view the networking configuration that the guest sees

~~~bash
[centos@centos-workshop ~]# ip a s eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:b1:ce:00:00:05 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.2/24 brd 10.0.2.255 scope global dynamic noprefixroute eth0
       valid_lft 86309257sec preferred_lft 86309257sec
    inet6 fe80::57:a8ff:fe00:2/64 scope link
       valid_lft forever preferred_lft forever
~~~

*Wait*, why is this IP address **10.0.2.2** inside of the guest?! Well, in OpenShift Virtualization, every VM has the "same IP" inside the guest, and the hypervisor is bridging (**masquerading**) the pod network into the guest via a tap device. So don't be alarmed when you see the IP address being different here.

Now if we curl the IP address on the pod network (making sure you change this IP for the one that your VM is using on the pod network, **not** **10.0.2.2**.)
               
~~~bash
$ curl http://10.128.2.78
~~~

Which should show the following

~~~
Server address: 10.0.2.2:80
Server name: fedora-workshop
Date: 29/Jul/2022:15:12:04 +0000
URI: /
Request ID: aefdsf32e4622das5e604b6454359ec04234
~~~

Note the "server address" being the **10.0.2.2** address.

> Let's go to our OC CLI client  

## Exposing the VM to the outside world

In this step we're going to interface our Fedora VM to the outside world using OpenShift/Kubernetes networking constructs, namely services and routes, demonstrating that whilst this is a VM, it's just "another" type of workload as far as OpenShift is concerned, and the same principles should be able to be applied. This step makes our VM available via the OpenShift ingress service and you should be able to hit our Fedora VM from the internet. As validated in the previous step, our VM has NGINX running on port 80, so let's use the `virtctl` utility, a CLI tool for managing OpenShift Virtualization above what `oc` provides, to expose the virtual machine instance on that port. 

First `expose` it on port 80 and create a service (an entrypoint) based on our Fedora VM

```execute-1
virtctl expose virtualmachineinstance < fedora-workshop > --name fedora-workshop-http-service --port 80
```

This should create a new service for us

~~~bash
Service fedora-workshop-http-service successfully exposed for virtualmachineinstance fedora-workshop
~~~

Let's take a look at that service

```execute-1
oc get svc/fedora-workshop-http-service
```

You'll see that we have a new service with a cluster IP associated, listening on port 80:

~~~bash
NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
fedora-workshop-http-service   ClusterIP   172.30.73.174   <none>        80/TCP    93s
~~~

Next we create a route for our service, this will associate a URL that we can connect to

```execute-1
oc create route edge --service=fedora-workshop-http-service
```

A new route will be created

~~~bash
route.route.openshift.io/fedora-workshop-http-service created
~~~

And view the route

```execute-1
oc get routes
```

Here you can see the output of our route with the hostname generated based on our cluster

~~~bash
NAME                           HOST/PORT                                                                   PATH   SERVICES                       PORT    TERMINATION   WILDCARD
fedora-workshop-http-service   fedora-workshop-http-service-workshop.apps.almog-elfassy.internal           fedora-workshop-http-service   <all>   edge          None
~~~

You can now visit the endpoint at https://fedora-workshop-http-service-workshop.apps.almog-elfassy.internal in a new browser tab and find the NGINX server from your Fedora based VM - you should see the same content that we curl'd in a previous step, just now it's exposed on the internet

> **NOTE**: If you get an "Application is not available" message, make sure that you're accessing the route with **https** - the router performs TLS termination for us, and therefore there's not actually anything listening on port 80 on the outside world, it just forwards 443 (OpenShift ingress) -> 80 (pod network).

We've successfully exposed our VM externally to the internet via the pod network, just like any other containerised application.

### Let's Clone the Fedora VM 

Now that we know our Fedora VM is working, we need to shutdown the VM so we can clone it without risking filesystem corruption. We'll use the `virtctl` command to help us with this as it saves us logging back into the VM by interacting via the guest agent

```execute-1
virtctl stop fedora-workshop
```

The VM is now scheduled to stop

~~~bash
VM fedora-workshop was scheduled to stop
~~~

Next, check the list of `vm` objects. It's now marked as `Stopped`

```execute-1
oc get vm
```

~~~bash
NAME              AGE   STATUS     READY
centos-workshop   89m   Running    True
fedora-workshop   56m   Stopping   False
~~~

Now that we've got a working virtual machine with a test workload we're ready to actually clone it. This will demonstrate that the built-in cloning utilities work and that the cloned machine shares the same workload. 

There are a couple of ways of doing this, but first we'll use the CLI to do it. For this method we clone the underlying storage volume by creating a PV (persistent volume) to clone into. This is done with a special resource called a `DataVolume`. This custom resource type is provide by CDI. DataVolumes orchestrate import, clone, and upload operations and help the process of importing data into a cluster. DataVolumes are integrated into OpenShift Virtualization.

The volume we are creating is named `fedora-clone` and we'll be pulling the data from the volume ("source") `fedora-workshop`


```execute-1
cat << EOF | oc apply -f -
apiVersion: cdi.kubevirt.io/v1alpha1
kind: DataVolume
metadata:
  name: fedora-clone
spec:
  source:
    pvc:
      namespace: workshop
      name: fedora-workshop
  pvc:
    volumeMode: Block
    storageClassName: ocs-storagecluster-ceph-rbd
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 30Gi
EOF
```

The DataVolume is created

~~~bash
datavolume.cdi.kubevirt.io/fedora-clone created
~~~

Usually, a clone goes through a number of stages, and you can watch the progress through `CloneScheduled` and `CloneInProgress` phases. However in our case we're using OpenShift Container Storage which makes an instant clone of a volume within the storage platform and doesn't require this process. 

You'll be able to view the status of the clone and its `PHASE` as "**Succeeded**". We are also able to view all PVCs including the new clone

```execute-1
oc get dv/fedora-clone pvc/fedora-clone
```

This should show both objects

~~~bash
NAME                                      PHASE       PROGRESS   RESTARTS   AGE
datavolume.cdi.kubevirt.io/fedora-clone   Succeeded                         78s

NAME                                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS                  AGE
persistentvolumeclaim/fedora-clone   Bound    pvc-a4c06199-cff1-4955-930a-1fee78e1d597   30Gi       RWX            ocs-storagecluster-ceph-rbd   75s
~~~

### Start the cloned VM

We can now start up a new VM using the cloned PVC. First create a new definition of a `VirtualMachine` to house the VM and start it automatically (`running: true`)

```execute-1
cat << EOF | oc apply -f -
apiVersion: kubevirt.io/v1alpha3
kind: VirtualMachine
metadata:
  name: fedora-clone
  labels:
    app: fedora-clone
    os.template.kubevirt.io/fedora34: 'true'
    vm.kubevirt.io/template-namespace: openshift
    workload.template.kubevirt.io/server: 'true'
spec:
  running: true
  template:
    metadata:
      labels:
        vm.kubevirt.io/name: fedora-clone
    spec:
      domain:
        cpu:
          cores: 1
          sockets: 1
          threads: 1
        devices:
          disks:
            - bootOrder: 1
              disk:
                bus: virtio
              name: disk0
            - disk:
                bus: virtio
              name: cloudinitdisk
          interfaces:
            - name: default
              masquerade: {}
          networkInterfaceMultiqueue: true
          rng: {}
        machine:
          type: pc-q35-rhel8.4.0
        resources:
          requests:
            memory: 2Gi
      evictionStrategy: LiveMigrate
      hostname: fedora-clone
      networks:
        - name: default
          pod: {}
      terminationGracePeriodSeconds: 0
      volumes:
        - name: disk0
          persistentVolumeClaim:
            claimName: fedora-clone
        - cloudInitNoCloud:
            userData: |-
              #cloud-config
              user: fedora
              password: aelfassy
              chpasswd: { expire: False }
          name: cloudinitdisk 
EOF
```

This shows the VM creation

~~~bash
virtualmachine.kubevirt.io/fedora-clone created
~~~

After a few minutes you should see the new virtual machine running

```execute-1
oc get vm
```

You will have one running (the clone), and one stopped (the original):

~~~bash
NAME              AGE    STATUS    READY
centos-workshop   101m   Running   True
fedora-clone      23s    Running   True
fedora-workshop   67m    Stopped   False
~~~

This machine should also get an IP address after a few minutes - it won't be the same as the original VM as the clone was given a new MAC address, you may need to be patient here until it shows you the IP address of the new VM

```execute-1
oc get vmi
```

~~~bash
NAME            AGE   PHASE     IP            NODENAME                                                READY
fedora-clone    88s   Running   10.128.2.89   workshop-n54ln-worker-c-x98pv.c.almog-elfassy.internal  True
centos-workshop 101m  Running   10.128.2.81   workshop-n54ln-worker-c-x98pv.c.almog-elfassy.internal  True
~~~

This machine will also be visible from the OpenShift Virtualization console, which you can navigate to using the top "**Console**" button, 
going into the "**Workloads**" --> "**Virtualization**" --> "**fedora-clone**"

### Test the clone

Like before, we should be able to just directly connect to the VM on port 80 via `curl` and view our simple NGINX based application responding. Let's try it! 

First access to our **VM client** (centos) by console

<img width="597" alt="Screen Shot 2022-07-29 at 15 02 50" src="https://user-images.githubusercontent.com/64369864/181754474-bdbc8c81-bdfe-44f2-83a4-ccd07d5ec5ce.png">

Check the simple NGINX based application by `fedora-clone` pod IP (oc get vmi)

~~~copy
$ curl http://10.128.2.89
~~~

Which should show similar to the following, if our clone was successful

~~~bash
Server address: 10.0.2.2:80
Server name: fedora-clone
Date: 30/Jul/2022:15:12:04 +0000
URI: /
Request ID: 30d16f4250df0d0d82ec2af2ebb60728
~~~

Our VM was cloned! At least the backend storage volume was cloned and we created a new virtual machine from it. Now you're probably thinking "wow, that was a lot of work just to clone a VM", and you'd be right! There's a much more simple workflow via the UI, and one that copies over all of the same configuration without us having to define a new VM ourselves. Let's first delete our clone, and then we'll move onto re-cloning the original via the UI:

```execute-1
oc delete vm/fedora-clone dv/fedora-clone pvc/fedora-clone
```

This should delete all objects at the same time

~~~bash
virtualmachine.kubevirt.io "fedora-clone" deleted
datavolume.cdi.kubevirt.io "fedora-clone" deleted
persistentvolumeclaim "fedora-clone" deleted
~~~

Now, if we navigate to the OpenShift Console, and ensure that we're in the list of Virtual Machines by selecting "**Workloads**" --> "**Virtualization**", we should see our "*fedora-workshop*" VM as stopped

<img width="634" alt="Screen Shot 2022-07-30 at 23 53 50" src="https://user-images.githubusercontent.com/64369864/181995863-87932a99-48af-488a-b05b-4dd627da629d.png">

Select "*fedora-workshop*" and then from the "**Actions**" drop-down on the right hand side, select "**Clone Virtual Machine**". This will bring up a new window where we can confirm our requirements

<img width="968" alt="Screen Shot 2022-07-30 at 23 56 15" src="https://user-images.githubusercontent.com/64369864/181995915-364cfaf5-a02e-42b6-90d9-c73ae5a67717.png">

We'll leave the defaults here, but make sure to select "**Start virtual machine on clone**" as this will ensure that our freshly cloned VM is automatically started for us. When you're ready, select the blue "**Clone Virtual Machine**" button at the bottom; this will create an identical virtual machine for us, just with a new name, "*fedora-workshop-clone*".

As soon as this happens, a new virtual machine will be created and started for you. You can see this in "**Workloads**" --> "**Virtualization**" or via the CLI

```execute-1
oc get vm
```

Again, we have two VM's, our original stopped, and our new clone running:

~~~bash
NAME                    AGE    STATUS    READY
centos-workshop         128m   Running   True
fedora-workshop         94m    Stopped   False
fedora-workshop-clone   60s    Running   True
~~~

Let's check our VMI list

```execute-1
oc get vmi
```

Here our running VM is showing with our new IP address

~~~bash
NAME                     AGE   PHASE     IP            NODENAME                                                READY
fedora-workshop-clone    88s   Running   10.128.2.94   workshop-n54ln-worker-c-x98pv.c.almog-elfassy.internal  True
centos-workshop          101m  Running   10.128.2.81   workshop-n54ln-worker-d-x98pv.c.almog-elfassy.internal  True
~~~

Like before, we should be able to confirm that it really is our clone by use our client VM (centos)

~~~bash
$ curl http://10.128.2.94
~~~

Which should show something similar to this:

~~~bash
Server address: 10.0.2.2:80
Server name: fedora-workshop
Date: 31/Jul/2022:00:26:05 +0000
URI: /
Request ID: a966basdr321dsadfl931dfdsaf231299df
~~~

There we go! We have successfully cloned a VM via the CLI (and backend DataVolume) as well as used the UI to do it for us. Let's clean up our clone, and our original before proceeding (don't delete the PVC, we'll use it in a later lab)

```execute-1
oc delete vm/fedora-workshop-clone
oc delete vm/centos-workshop
```

For the next step, start the fedora-workshop VM. We'll use the `virtctl` command to help us with this

```execute-1
virtctl start fedora-workshop
```


