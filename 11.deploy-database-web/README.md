In this section we will deploy and connect a MongoDB database where the`nationalparks` application will store the location information. This time we are going to deploy the MongoDB application in a Virtual Machine by leveraging OpenShift Virtualization; that way we're demonstrating the capability for OpenShift to connect multiple types of workloads, regardless of whether they're containerised, or virtualised.


### 1.  Search for the Virtual Machine Template


In this module we will create a MongoDB instance from a *Template*, again based on a template that we've already pre-loaded, which contains all the necessary Kubernetes resources and configuration to deploy and run MongoDB in a VM, based on CentOS 8 - as part of a VM template that we created in an earlier step.

For this, make sure that you're still in the web console view, and if you are in the in the *Administrator* perspective, switch to Developer perspective and go to the **parksmap-< your name >-namespace** project. 

- From the left menu, click *+Add*. You will see a screen where you have multiple options to deploy application. 

- Then Click *All Services* and in the *Search* text box and enter *mongo* to find the MongoDB VM template. 

<br/>

<img width="912" alt="Screen Shot 2022-08-01 at 15 39 20" src="https://user-images.githubusercontent.com/64369864/182149721-ab7a2b00-598f-4a69-9215-b6b5c49cfd8a.png">

![image](https://user-images.githubusercontent.com/64369864/182149837-edc0def4-3379-4700-8420-01eb0115ba61.png)

<br/>

### 2. Instantiate the Virtual Machine Template

In order to instantiate the template, first click on the `MongoDB Virtual Machine` template to open the popup menu 
and then click on the *Instantiate Template* button as you did when you deployed the parksmap application components.
This will open a dialog that will allow you to configure the template. This template allows you to configure the following parameters:

- *MongoDB Application Name*
- *Virtual Machine Username*
- *Virtual Machine User Password*
- *Database Name*
- *Database Username*
- *Database Admin Password*
  

Enter ***mongodb-nationalparks*** in  **MongoDB Application Name** field and leave other parameter values default.

 <br/>

![image](https://user-images.githubusercontent.com/64369864/182149987-891e36bf-6810-4ece-a02f-f42c4184660e.png)

 <br/>

> **NOTE**: Make sure that you update the MongoDB Application Name before clicking "Create".

Next click the blue ***Create*** button. 

You will be directed to the *Topology* page, where you should see the visualisation for the `mongodb-nationalparks` virtual machine in the `workshop` application, i.e. we're extending the existing application that we have. OpenShift creates both *VirtualMachine* and *Service* objects. The `nationalparks` backend application will use this *mongodb-nationalparks service* to communicate with MongoDB.

### 3. Verify the Database Service in Virtual Machine  

It will take some time MongoDB VM to start and initialise. You can check the status of the VM in the Web Console by clicking on the VM details in the Topology View or execute following command in the terminal 

![image](https://user-images.githubusercontent.com/64369864/182150556-ff3c5b79-8fe6-40a5-863b-e3d950097826.png)

```
oc project < parksmap-project-namespace >
```

Make sure the workload is created and in a `running` state by following commands

```
oc get vm
```

Which, after a few minutes should show

~~~bash
NAME                    AGE   STATUS     READY
mongodb-nationalparks   45s   Running    True
~~~

Once the MongoDB Virtual Machine is in a Running state, open the *Virtual Machine Console* by switching to the "**Administrator**" perspective in the top left hand corner, and then navigating to "**Workloads**" --> "**Virtualization**" and selecting "**mongodb-nationalparks**". Once there, select the "**Console**" tab and you should be able to see the virtual machine console. On the login screen, enter the following credentials

~~~bash
Login: < mongodb-vm-username >
Password: < mongodb-vm-password >
~~~

<img width="1205" alt="Screen Shot 2022-08-01 at 15 46 54" src="https://user-images.githubusercontent.com/64369864/182151056-37403516-cbc0-4ac5-b98a-513bffe5ddfb.png">

<img width="793" alt="Screen Shot 2022-08-01 at 15 49 14" src="https://user-images.githubusercontent.com/64369864/182151419-a902f9a6-da9a-4559-a41f-a0c558b43902.png">


Check whether *mongod* service is running by executing following:

```execute-1
systemctl status mongod
```

Please verify whether *mongod* service is up and running as shown in the figure below. If you try and do this too quickly, MongoDB might not have started yet, and you may have to wait a few minutes.

 <br/>

![image](https://user-images.githubusercontent.com/64369864/182151611-cb853129-9baa-4ea7-81cf-65abaa87bf88.png)

 <br/>

### 4. Verify Nationalparks Application

Now that we have a database deployed, we can again visit the `nationalparks` web service to query for data:


[http://< nationalparks-parksmap-project-namespace >.< apps.master.aelfassy.com >/ws/data/all](http://nationalparks-parksmap-project-namespace.apps.master.aelfassy.com/ws/data/all)

And the result? It should just show this, an empty dataset
~~~bash
[]
~~~

<img width="798" alt="Screen Shot 2022-08-01 at 16 01 04" src="https://user-images.githubusercontent.com/64369864/182153608-7f7434dd-2e25-423a-bdaf-bba6c20abf70.png">

Where's the data? Think about the process you went through. You deployed the application and then deployed the database. Nothing actually loaded anything *INTO* the database, though. The application provides an endpoint to do just that, load in some data:

[http://< nationalparks-parksmap-project-namespace >.< apps.master.aelfassy.com >/ws/data/load](http://nationalparks-parksmap-project-namespace.apps.master.aelfassy.com/ws/data/load)

And the result?

~~~bash
Items inserted in database: 2893
~~~

<img width="798" alt="Screen Shot 2022-08-01 at 16 02 46" src="https://user-images.githubusercontent.com/64369864/182153913-8564a129-a693-44aa-a6ab-04cb7e6ef4f6.png">

If you then go back to `/ws/data/all` you will see tons of JSON data now. This data will be visualised on the map if you check your browser now: [http://< parksmap-parksmap-project-namespace >.< apps.master.aelfassy.com >](http://parksmap-parksmap-project-namespace.apps.master.aelfassy.com)

 You'll notice that the national parks suddenly are showing up as below. 
 <br/> 

<img width="1429" alt="Screen Shot 2022-08-01 at 16 06 14" src="https://user-images.githubusercontent.com/64369864/182154501-05a3d463-bb07-4934-89fd-8f986ecb4909.png">

### 5. Understand the MongoDB Virtual Machine Template

As you've seen so far, the web console and the templates makes it very easy to deploy things onto OpenShift, regardless of the type of workload. When we deploy the database virtual machine, we pass in some values for configuration. These values are used to set the username, password, name of the database, etc. Let's have a look at the template definition. Switch over to the terminal view and execute the following command to find the template

```execute-1
oc get templates -n openshift| grep mongodb-vm-template
```

This should list the MongoDB Virtual Machine Template we are looking for:

~~~bash
mongodb-vm-template  
~~~

Now let's check the template definition

```execute-1
oc get template mongodb-vm-template -n openshift -o yaml
```

There are many details, but let's focus on the `cloudInitNoCloud` section. This is the part where put the instructions to initialise the Virtual Machine, because remember, this VM would have started out as a vanilla template, with zero configuration. The cloud-init tool provides us with a way of injecting deploy-time instructions to our VM

~~~yaml
(...)

- cloudInitNoCloud:
  userData: |-
    #cloud-config
    user: ${VM_USER_NAME}
    password: ${VM_PASSWORD}
    chpasswd: { expire: False }
    yum_repos:
      mongodb-org-3.6:
        baseurl: https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.6/x86_64/
        enabled: true
        gpgcheck: true
        gpgkey: https://www.mongodb.org/static/pgp/server-3.6.asc
        name: MongoDB Repository
    packages:
      - bash-completion
      - mongodb-org
    runcmd:
      - systemctl daemon-reload
      - systemctl enable mongod.service
      - systemctl start mongod.service
      - >
        mongo admin --host localhost --eval "db.createUser({user: 'root', pwd: '${MONGODB_ADMIN_PASSWORD}', roles: [{role: 'root', db: 'admin'}]});"
      - >
        mongo admin --host localhost --eval "db.getSiblingDB('mongodb').createUser({user: '${MONGODB_USER}', pwd: '${MONGODB_PASSWORD}', roles: [{role: 'dbOwner', db: '${MONGODB_DATABASE}'}]});"
      - >
        sed -i -e 's/bindIp: 127.0.0.1/bindIp: 0.0.0.0/g' /etc/mongod.conf
      - >
        sed -i -e 's/^#security:/security:/g' /etc/mongod.conf
      - >
        sed -i -e '/^security:/a \  authorization: enabled' /etc/mongod.conf
      - systemctl restart mongod.service
name: cloudinitdisk

...
~~~

In the above output, you'll notice that there are placeholders for variables, if we check the VirtualMachine object now...

```execute-1
 oc get vm mongodb-nationalparks -n < parksmap-project-namespace > -o yaml
```

... we will see that when we instantiate the template, OpenShift replaces the parameters with the values provided :

~~~yaml
...
 - cloudInitNoCloud:
    userData: |-
      #cloud-config
      user: redhat
      password: openshift
      chpasswd: { expire: False }
      yum_repos:
        mongodb-org-3.6:
          baseurl: https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.6/x86_64/
          enabled: true
          gpgcheck: true
          gpgkey: https://www.mongodb.org/static/pgp/server-3.6.asc
          name: MongoDB Repository
      packages:
        - bash-completion
        - mongodb-org
      runcmd:
        - systemctl daemon-reload
        - systemctl enable mongod.service
        - systemctl start mongod.service
        - >
          mongo admin --host localhost --eval "db.createUser({user: 'root', pwd: 'mongodb', roles: [{role: 'root', db: 'admin'}]});"
        - >
          mongo admin --host localhost --eval "db.getSiblingDB('mongodb').createUser({user: 'mongodb', pwd: 'mongodb', roles: [{role: 'dbOwner', db: 'mongodb'}]});"
        - >
          sed -i -e 's/bindIp: 127.0.0.1/bindIp: 0.0.0.0/g' /etc/mongod.conf
        - >
          sed -i -e 's/^#security:/security:/g' /etc/mongod.conf
        - >
          sed -i -e '/^security:/a \  authorization: enabled' /etc/mongod.conf
        - systemctl restart mongod.service
  name: cloudinitdisk
...
~~~

OpenShift utilises `cloud-init` which is a widely adopted project used for early initialisation of a VM. Used by cloud providers such as AWS and GCP, `cloud-init` has established itself as the defacto method of providing startup scripts to VMs. Cloud-init documentation can be found here: [https://cloudinit.readthedocs.io/en/latest/](https://cloudinit.readthedocs.io/en/latest/), if you'd like to better understand its capabilities.

OpenShift Virtualization supports cloud-init's "NoCloud" and "ConfigDrive" datasources which involve injecting startup scripts into a VM instance through the use of an ephemeral disk. VMs with the cloud-init package installed will detect the ephemeral disk and execute custom userdata scripts at boot. Other than cloud-init, OpenShift Virtualization also supports `SysPrep` which is an automation tool for Windows that automates Windows installation, setup, and custom software provisioning. 

You can automate Windows virtual machine setup by uploading answer files in XML format in the Advanced → SysPrep section of the "Create Virtual Machine" from template wizard, but we won't explore that in this lab.

