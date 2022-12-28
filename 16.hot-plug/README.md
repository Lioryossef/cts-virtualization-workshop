### Background: Hot-plugging virtual disks
It is expected to have **dynamic reconfiguration** capabilities for VMs today, such as CPU/Memory/Storage/Network hot-plug/hot-unplug. Although these capabilities have been around for the traditional virtualisation platforms, it is a particularly challenging feature to implement in a **Kubernetes** platform because of the Kubernetes principle of **immutable pods**, where once deployed they are never modified. If something needs to be changed, you never do so directly on the Pod. Instead, you’ll build and deploy a new one that has all your needed changes baked in.

OpenShift Virtualization strives to have these dynamic reconfiguration capabilities for VMs although it's a Kubernetes-based platform. In the 4.9 release, hot-plugging virtual disks to a running virtual machine is supported as a [Technology Preview](https://access.redhat.com/support/offerings/techpreview) feature, so as a VM owner, you are able to attach and detach storage on demand.

### Exercise: Hot-plugging a virtual disk using the web console
In OpenShift Virtualization it's possible to hot-plug and hot-unplug virtual disks without stopping your virtual machine. This capability is helpful when you need to add storage to a running virtual machine without incurring down-time. When you hot-plug a virtual disk, you attach a virtual disk to a virtual machine instance while the virtual machine is running. When you hot-unplug a virtual disk, you detach a virtual disk from a virtual machine instance while the virtual machine is running. Only data volumes and persistent volume claims (PVCs) can be hot-plugged and hot-unplugged. You cannot hot-plug or hot-unplug *container* disks.

In this exercise, let's attach a new 2GB disk to one of our MongoDB database VM's by using the web console
<table>
  <tr>
    <td>

1. Click **Virtualization** from the side menu.
   
2. Click the **Virtual Machines** tab.
   
3. Select **your virtual machine** (In my example `mongodb-nationalparks` virtual machine) to open its **Overview** screen.

4. On the **Disks** tab, click **Add Disk**.

5. In the **Add Disk** window, fill in the information for the virtual disk that you want to hot-plug. **Be sure to review the image provided to ensure you set the values correctly.**

6. Click **Add**.
    </table>

![image](https://user-images.githubusercontent.com/64369864/183288330-6778c2e4-72fe-4021-9615-6803136c113d.png)

![image](https://user-images.githubusercontent.com/64369864/183288347-505f5cba-ccad-4625-a8ac-f5e1e27d6337.png)
  
Once you click Add to attach new disk, a new vm disk is automatically provisioned using the selected storage class, which is ceph-rbd in this exercise, and attached to the running virtual machine. You can see the new disk in the Disks tab of the virtual machine.

<img width="1007" alt="Screen Shot 2022-08-07 at 14 29 45" src="https://user-images.githubusercontent.com/64369864/183288405-fabd9390-a393-498e-81e7-ff9c8484ef83.png">
  
To verify if the new 5GB disk is recognised and ready to use by the guest operating system, let's connect the console of our virtual machine and list block devices:

1. Click **Virtualization** from the side menu.
   
2. Click the **Virtual Machines** tab.
   
3. Select  **your virtual machine** (In my example `mongodb-nationalparks` virtual machine) to open its **Overview** screen.

4. Navigate to the "**Console**" tab. You'll be able to login with "**redhat/openshift**", noting that you may have to click on the console window for it to capture your input.

5. Once you're in the virtual machine, run the *lsblk* command to list block devices recognised by the operating system.
  
```copy
sudo lsblk
```

<img width="703" alt="Screen Shot 2022-08-07 at 14 31 42" src="https://user-images.githubusercontent.com/64369864/183288464-0d7941f7-16d7-4018-acf2-23e522c3a1ff.png">
  
> **NOTE**: This showed up as "**sda**", as the default interface is "**scsi**" - if we'd have chosen "virtio" this would have been a "**vd***" device.

### Exercise: Expand the VM's disk

OpenShift allows users to easily resize an existing PersistentVolumeClaim (PVC) objects. You no longer have to manually interact with the storage backend or delete and recreate PV and PVC objects to increase the size of a volume. Shrinking persistent volumes is not supported. In this exercise, let's resize our hot-plugged 5GB disk to 7GB by using the web console.

<table>
  <tr>
    <td>

1. Click **Workloads** → **Virtualization** from the side menu.
   
2. Click the **Virtual Machines** tab.
   
3. Select  **your virtual machine** (In my example `mongodb-nationalparks` virtual machine) to open its **Overview** screen.

4. On the **Disks** tab, click the **PVC name** of the `PersistingHotplug` disk.

5. In the **PersistentVolumeClaims** window, click **Actions** → **Expand PVC**

6. In the **Expand PersistentVolumeClaim** pop-up window, set the **Total Size** as **3 GiB**

7. Click **Expand**.
   
    </table>

![image](https://user-images.githubusercontent.com/64369864/183288545-dd702de7-fea6-4863-95a8-cadfb17d7003.png)

![image](https://user-images.githubusercontent.com/64369864/183288564-a60f5e49-29f6-4b93-a215-dcf8cedac880.png)

An expanded disk's **new** size is automatically recognised by the OS. To verify it, let's connect the console of our virtual machine and check the list block devices again.

Size of the hot-plugged disk should be as 3GB instead of 2GB.

1. Click **Virtualization** from the side menu.
   
2. Click the **Virtual Machines** tab.
   
3. Select **your virtual machine** (In my example `mongodb-nationalparks` virtual machine) to open its **Overview** screen.

4. Navigate to the "**Console**" tab. You'll be able to login with "**redhat/openshift**" (if you're not already logged in), noting that you may have to click on the console window for it to capture your input.

5. Once you're in the virtual machine, run the lsblk command to list block devices recognized by the operating system.
```execute
sudo lsblk
```
  
<img width="616" alt="Screen Shot 2022-08-07 at 14 38 21" src="https://user-images.githubusercontent.com/64369864/183288716-a1d875ac-602d-4c63-a2f1-cb427c9f699a.png">

> As we see, the OS sees the new disk as 3GB.

### Exercise: Hot-unplugging a virtual disk using the web console
It's possible to hot-**un**plug virtual disks when you want to remove them without stopping your virtual machine or virtual machine instance. This capability is helpful when you need to remove storage from a running virtual machine without incurring down-time. When you hot-unplug a virtual disk, you detach a virtual disk from a virtual machine instance while the virtual machine is running. Only data volumes and persistent volume claims (PVCs) can be hot-unplugged. In this exercise, let's detach the disk that we have hot-plugged in the previous exercise from our MongoDB database VM by using the web console:

<table>
  <tr>
    <td>

1. Click **Virtualization** from the side menu.
   
2. Click the **Virtual Machines** tab.
   
3. Select **your virtual machine** (In my example `mongodb-nationalparks` virtual machine) to open its **Overview** screen.

4. Click the **Disks** tab. The page displays a list of disks attached to the virtual machine.

5. Click the Options menu of the **disk** named `disk-0` which is marked as `PersistingHotplug`

6. Click **Delete**.
   
7. In the confirmation pop-up window, click **Detach** to hot-unplug the disk.
    </table>

<img width="1010" alt="Screen Shot 2022-08-07 at 14 43 43" src="https://user-images.githubusercontent.com/64369864/183288920-c226fb4c-3287-4d77-86f0-2f21de0a8bbc.png">

![image](https://user-images.githubusercontent.com/64369864/183288938-f22d04b5-c87f-49a6-a170-e66dccb212c9.png)
  
Once you click **Detach** to hot-unplug the disk, it's detached from the running virtual machine and guest operating system automatically recognises the event. To verify if the 7G disk removal is recognised by the guest operating system, let's connect the console of our virtual machine and list block devices once again. The disk should no longer be listed by the guest operating system.

1. Click **Virtualization** from the side menu.
   
2. Click the **Virtual Machines** tab.
   
3. Select **your virtual machine** (In my example `mongodb-nationalparks` virtual machine) to open its **Overview** screen.

4. Navigate to the "**Console**" tab. You'll be able to login with "**redhat/openshift**", noting that you may have to click on the console window for it to capture your input.

5. Once you're in the virtual machine, run the lsblk command to list block devices recognised by the operating system.
```execute
sudo lsblk
```
<img width="559" alt="Screen Shot 2022-08-07 at 14 45 39" src="https://user-images.githubusercontent.com/64369864/183288989-2d3125df-2447-4882-9b9d-05dc1f9aee50.png">

That's it for hot-plugging and expanding virtual disks - we've hot-plugged a new 2GB disk to our MongoDB database virtual machine using OpenShift web console, expanded its size to 3GB and finally hot-unplugged it from our virtual machine. 
