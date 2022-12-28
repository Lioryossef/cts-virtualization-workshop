Now that you've had a chance to dive into how OpenShift virtualisation works at a very deep level, let's spend a little time looking at an equally important component of the OpenShift virtualisation experience: the User Interface (UI). As with all Red Hat products OpenShift virtualisation offers a rich CLI allowing administrators to easily script repetitive actions, dive deep into components, and create almost infinite levels of configuration. In these labs you've use both virtctl, the OpenShift virtualisation client, and oc, the OpenShift Container Platform client, to do many tasks.

However, the need for an easy to use, developer friendly User Experience is also a key value for OpenShift and OpenShift virtualisation. Let's take a look at some of the outcomes and actions from the previous labs and how they can be achived and/or reviewed from within OpenShift's exceptional UI.

### Connecting to the console
 
<img width="1412" alt="Screen Shot 2022-07-31 at 0 52 02" src="https://user-images.githubusercontent.com/64369864/181997441-da229f61-309e-4c58-ade1-47ae0afcb546.png">

### Virtual Machines in OpenShift!
Once OpenShift Virtualization is installed an Option called Virtualization will appear

![image](https://user-images.githubusercontent.com/64369864/181998274-99d93802-6fca-4169-a491-f8266e33cf72.png)

> NOTE: This new option contains all the OpenShift Virtualization finctionality of the previous two menu items in one place. Any previous representations of the console used within this lab are being upated, and should be used interchangeably.

You can then choose the your project and "**Virtualization**" --> "**Virtual Machines**" menu and you will see the running VMs from the labs. 

Click on the link for the "**fedora-workshop**" instance. You can see some of the familiar features of this instance, such as the IP, node, OS, and more.

<img width="469" alt="Screen Shot 2022-07-31 at 1 14 27" src="https://user-images.githubusercontent.com/64369864/182001941-32a46cbc-655a-4312-bd46-7345af1d62df.png">

![image](https://user-images.githubusercontent.com/64369864/182001952-60f883fb-565a-4e9a-8178-cbc968ddff3f.png)

Across the top is a helpful menu allowing you deeper access to administer and access the running VM. Choose `YAML` to view, edit, and reload the template defining this instance. `Consoles` give you access, via both VNC and Serial to the instance's console for easy login. `Events` display, of course, important events for the instance. With `Network Interfaces` we can see the name, model, binding, etc for the instances network connections. Under `Disks` we can see the storage class used and the type of interface. Be sure to look at the different entries for different types of virtual machines. You should be able to directly connect the names, values, and information to the services and objects you created before.

But it's not just the VM instances available. OpenShift virtualisation is merely an extension to OpenShift and all the components we created and used for our VMs are part of the OpenShift experience just like they are for pods. On the menu on the left choose "**Networking**" and select "**Services**"

![image](https://user-images.githubusercontent.com/64369864/182001985-8e8dd7d9-b34d-4830-8875-437127c9f33d.png)

Here you can see the networking services we created for our Fedora host for http, ssh, and NodePort SSH. Try drilling down on the Fedora Service. You should see all the features of the service including Cluster IP, port mapping, and routing. From the actions menu in the upper right corner you can edit these values directly and update your VM's service directly from the UI, in the same was as you can do with pods.

Next click on the Pods menu item

![image](https://user-images.githubusercontent.com/64369864/182002020-e0232bf5-7472-4ef0-a023-6cf8d0e86268.png)

Here we can see the pod that is running the VM instance (the "launcher" pod we learned about previously) as well as the host it is running on

![image](https://user-images.githubusercontent.com/64369864/182002026-e825ee59-057d-4fde-a30b-3d009435fab7.png)

Choose the link for the launcher pod and we can really see the components we built behind the pod. It's important again to note that from here we have access like any other pod in OpenShift. We can use the menu across the top to view the Pod's YAML, Logs, and even access the pod's shell (note this is NOT the VM's shell or console, this is for the pod which **runs** the VM).

Be sure to scroll down on this screen to the Volumes section.

<img width="1394" alt="Screen Shot 2022-07-31 at 1 23 09" src="https://user-images.githubusercontent.com/64369864/182002109-b01e18e5-9b5a-496e-a42e-3292339d1f2a.png">

We are able to see and edit all the features of the PVC we assigned for this pod.

The same is true for all components of our VM's and VMI's from the labs. If you choose any of the storage items from the now-highlighted storage menu on the left you will see. Try choosing "Storage" > "Persistent Volumes" to show the various PVs we created. Continue to "Persistent Volume Claims" and "Storage Classes" to complete the picture.

<img width="847" alt="Screen Shot 2022-07-31 at 1 27 18" src="https://user-images.githubusercontent.com/64369864/182002194-2f63f40c-4b79-4d92-94fa-1c3a6a534676.png">

And of course, as we did in the previous steps of the workshop, we can create new VMs, connect by console, etc.

![image](https://user-images.githubusercontent.com/64369864/182002296-eaa96fb6-190c-4017-ab5a-c106ffb982bc.png)




