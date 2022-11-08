In this section we will deploy an additional MongoDB database in a VM, called `mlbparks`, which will store the location of Major League Baseball stadiums; this will provide a secondary data source for our visualisation application (parksmap). This time we are going to deploy the MongoDB Virtual Machine with using command line.

### 1. Creating MongoDB Virtual Machine

Make sure that you're in the "Terminal" view on the lab guide and switch to *%parksmap-project-namespace%* project by executing following command, ignoring any errors that tell you that you're already in that project:

```execute-1
oc project < parksmap-project-namespace >
```

You should see 

~~~bash
Now using project "parksmap-project-namespace" on server "https://api.workshop.aelfassy.com:6443".
~~~

And then run the following command to instantiate the template, overriding the MongoDB Application name:

```execute
oc process mongodb-vm-template \
	-p MONGODB_APPLICATION_NAME=mongodb-mlbparks \
	-n openshift | oc create -f -
```

You should see 

~~~bash

service/mongodb-mlbparks created
virtualmachine.kubevirt.io/mongodb-mlbparks created
~~~

### 2. Verify the Database Service in Virtual Machine  

It will take some time for the MongoDB VM to start and initialise, just like the first time we did it. We can watch for the status by asking OpenShift for a list of VM's

```execute
oc get vm
```

We should now see two VM's running:

~~~bash
NAME                    AGE   STATUS    READY
mongodb-mlbparks        45s   Running   True
mongodb-nationalparks   22m   Running   True
~~~

Like before, this template is setup to utilise cloud-init to automatically bootstrap the VM with MongoDB and ensure that the service has started, so after a few minutes, the VM should be ready.

### 3. Verify Mlbparks Application

If you go back to the OpenShift web-console by selecting the "**Console**" button at the top of your screen, and switch back to the *Developer* perspective, you should be able to see all `parksmap application` components including the two MongoDB Virtual Machines:

 <br/>

<img width="750" alt="Screen Shot 2022-08-01 at 16 32 41" src="https://user-images.githubusercontent.com/64369864/182160292-6eb1538d-15f4-4714-906a-c5beefa9ffe8.png">

Now that we have the database deployed for `mlbparks` , we can again visit the mlbparks web service to query for existing data

[http://< mlbparks-parksmap-project-namespace >.< apps.master.aelfassy.com >/ws/data/all](http://mlbparks-%parksmap-project-namespace%.%cluster_subdomain%/ws/data/all)

And the result is empty as expected, as we've not yet uploaded the data for the MLB Park locations:

~~~bash
[]
~~~

<img width="623" alt="Screen Shot 2022-08-01 at 16 37 20" src="https://user-images.githubusercontent.com/64369864/182160393-b50366cf-6ec9-4156-b5e6-9de17c001705.png">

So to load the data, navigate to the following endpoint, which will automatically load in the data for us:

[http://< mlbparks-parksmap-project-namespace >.< apps.master.aelfassy.com >/ws/data/load](http://mlbparks-%parksmap-project-namespace%.%cluster_subdomain%/ws/data/load)

Now you should see the following

~~~bash
Items inserted in database: 30
~~~

<img width="655" alt="Screen Shot 2022-08-01 at 16 39 31" src="https://user-images.githubusercontent.com/64369864/182160670-0c15c0ab-2f0b-4bae-bdd2-c7d723826fc0.png">

If you return to your parksmap visualisation application in your browser you should be able to see the stadium locations in United States as well, and be able to switch between MLB Parks, and National Parks:

[http://< parksmap-parksmap-project-namespace >.< apps.master.aelfassy.com >](http://parksmap-%parksmap-project-namespace%.%cluster_subdomain%)

 <br/> 

<img width="1005" alt="Screen Shot 2022-08-01 at 16 35 29" src="https://user-images.githubusercontent.com/64369864/182160784-688f3912-4fbd-4dcc-a5f0-a4b57d724071.png">
