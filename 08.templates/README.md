## About virtual machine templates and boot sources

Virtual machines consist of a virtual machine definition and one or more disks that are backed by data volumes. Virtual machine templates enable you to create virtual machines using predefined virtual machine specifications, allowing for subsequent, repeated creation of similar virtual machines.

Every virtual machine template requires a **boot source**, which is a fully configured virtual machine disk image including configured drivers. Each virtual machine template contains a virtual machine definition with a pointer to the boot source. Each boot source has a predefined name and namespace. For some operating systems, a boot source is automatically provided. If it is not provided, then an administrator must prepare a custom boot source.

The namespace `openshift-virtualization-os-images` houses these templates and is installed with the OpenShift Virtualization Operator. Once the boot source feature is installed, you can create boot sources, attach them to templates, and create virtual machines from the templates.

Boot sources are defined by using a persistent volume claim (PVC) that is populated by uploading a local file, cloning an existing PVC, importing from a registry, or by URL. Attach a boot source to a virtual machine template by using the web console. After the boot source is attached to a virtual machine template, you create any number of fully configured ready-to-use virtual machines from the template.

Preconfigured Red Hat virtual machine templates are listed in the **Templates** tab within the **Virtualization** menu. These templates are available for different versions of Red Hat Enterprise Linux, Fedora, Microsoft Windows 10, and Microsoft Windows Servers. Each Red Hat virtual machine template is preconfigured with the operating system image, default settings for the operating system, flavor (CPU and memory), and workload type (server).

![image](https://user-images.githubusercontent.com/64369864/182002838-7204e2ac-e6c8-464b-8ef0-d6f319137e03.png)


The Templates tab displays four types of virtual machine templates:

- **Red Hat Supported** templates are fully supported by Red Hat.
- **User Supported** templates are Red Hat Supported templates that were cloned and created by users.
- **Red Hat Provided** templates have limited support from Red Hat.
- **User Provided** templates are Red Hat Provided templates that were cloned and created by users.

## Exercise: Importing a CentOS Stream 8 image as a boot source
There are four methods for selecting and adding a boot source in the web console:
- **Upload local file** (creates PVC)
- **Import via URL** (creates PVC)
- **Clone existing PVC** (creates PVC)
- **Import via Registry** (creates PVC)

In this exercise, we will use the **Import Via URL** method to import a CentOS Stream 8 qcow2 disk image from our internal http server. Whilst we've done this before with the CLI, this time we will use the OpenShift web console to designate an image and in the background a new PVC will be created and CDI will be used to import the disk image into the newly created PVC. 

The new PVC will then be set as the boot source of the selected CentOS 8 template and then cloned for each new virtual machine created using that template.

1. In the OpenShift console, click **Virtualization** from the side menu.
2. Click the **Templates** tab.
3. Click --> **Create** --> With Wizard
4. In the **Add boot source to template window**, select **Import via URL (creates PVC)** from the **Boot source type** drop down.
5. Input `Red Hat Enterprise Linux` qcow2 image from the [RHEL download page](https://access.redhat.com/downloads/content/479/ver=/rhel---8/8.5/x86_64/product-software). as the URL of the guest image into the **Import URL** field.
6. Set **Persistent Volume Claim size** as **30 GiB**. This will also be the size of the root disk of the VMs created by using this template.
7. Fill the Wizard. Your details should looks like this

![image](https://user-images.githubusercontent.com/64369864/182003034-6d492378-450e-406b-b810-22528d25965e.png)

![image](https://user-images.githubusercontent.com/64369864/182003410-f9050b77-1f23-4242-8c75-56e147574a39.png)

![image](https://user-images.githubusercontent.com/64369864/182003479-a473cb71-2cc9-45fb-a4d8-3e5f6fd7aec5.png)

![image](https://user-images.githubusercontent.com/64369864/182003484-17b09e07-b6fd-4072-8270-3ca880775662.png)

![image](https://user-images.githubusercontent.com/64369864/182003124-afbda605-5eb0-42f3-a3de-eb7153a0da7e.png)

![image](https://user-images.githubusercontent.com/64369864/182003128-bc08c171-f2c3-402a-90ca-622db1519bf2.png)

SSH key and service is an option for easy access
![image](https://user-images.githubusercontent.com/64369864/182003134-4341de4b-4bcd-475b-ba4c-8970166ed178.png)

![image](https://user-images.githubusercontent.com/64369864/182003139-e1c5cbe2-98ab-4f57-93d7-cc2f5245d66b.png)

![image](https://user-images.githubusercontent.com/64369864/182003182-94392b64-1456-47f9-8a4c-31a633a584d2.png)

8. Click **Save and import**.

Now you can verify that a boot source was added to the template

1. In the OpenShift console, click **Virtualization** from the side menu.
   
2. Click the Templates tab.

3. Confirm that the tile for `workshop-template` template displays a green checkmark.

That's it for adding boot sources to a template. We have imported a RHEL cloud disk image into a new PVC and attached that onto a RHEL virtual machine template which we will use to create new virtual machines. 

![image](https://user-images.githubusercontent.com/64369864/182003380-5fd79efa-ecce-40a9-9190-bf94c5c3bd92.png)

![image](https://user-images.githubusercontent.com/64369864/182003391-fd7ca2d0-8a8e-41d4-a16e-9da46d33604e.png)

![image](https://user-images.githubusercontent.com/64369864/182003400-f6707696-3205-4ef7-bf13-648377270662.png)

![image](https://user-images.githubusercontent.com/64369864/182003493-3e95e197-f77f-4105-9dee-fdb30311ce4b.png)

Once you click **Create** a new `PersistentVolumeClaim` of the specified size is automatically provisioned using the selected storage class, which is ceph-rbd in this exercise.

After creating the new `PersistentVolumeClaim`, a CDI (Containerized Data Importer) pod is started in the `workshop` namespace. This CDI pod downloads the specified disk image from the http URL and populates the newly created `PersistentVolumeClaim` with the rhel of that disk image. You can see this CDI pod by switching into the `workshop` project and selecting **Workloads** → **Pods** from the side menu.

![image](https://user-images.githubusercontent.com/64369864/182003552-90fad86a-2c3e-428a-bf6f-4970c78193fa.png)

You can also view the import progress by listing the data volumes in the `openshift-virtualization-os-images` namespace.

```execute
oc get datavolumes -n workshop
```

Which should show the following

~~~bash
NAME                                         PHASE              PROGRESS   RESTARTS   AGE
workshop-template-virtual-machine-rootdisk   ImportInProgress   37.48%                54s
~~~

Once the import progress is reached up to 100% and succeeded the VM will be ready 

![image](https://user-images.githubusercontent.com/64369864/182003600-49eb6e56-f790-42ec-ab42-76dd194fe4ae.png)

