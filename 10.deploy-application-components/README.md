In this lab, we will use the OpenShift Web Console to deploy the frontend and backend components of the ParksMap application, which comprises of one frontend web application, two backend applications and 2 databases:

- ParksMap frontend web application, also called `parksmap`, and uses OpenShift's service discovery mechanism to discover the backend services deployed and shows their data on the map.

- NationalParks backend application queries for national parks information (including their coordinates) that are stored in a MongoDB database. 

- MLBParks backend application queries Major League Baseball stadiums in the US that are stored in an another MongoDB database.

Parksmap frontend and backend components are shown in the diagram below
 <br/>

![Screen Shot 2022-07-31 at 12 50 35](https://user-images.githubusercontent.com/64369864/182020733-22fca50b-d0ac-4c3f-a807-430d48bd99da.png)

 <br/>

### 1. Creating the Project

As a first step, we need to create a project where ParksMap application will be deployed. You can create the project with the following command:

```execute
oc new-project parksmap-< Your name >-namespace
```

### 2.  Grant Service Account View Permissions

The ParksMap frontend application continously monitors the **routes** of the backend applications. This requires granting additional permissions to access OpenShift API to learn about other **Pods**, **Services**, and **Route** within the **Project**. 


```execute
oc policy add-role-to-user view -z default
```

You should see the following output

~~~bash
clusterrole.rbac.authorization.k8s.io/view added: "default"
~~~

The *oc policy* command above is giving a defined _role_ (*view*) to the default user so that applications in current project can access OpenShift API.

### 3. Navigate to the OpenShift Web Console

Select the blue "**Console**" button at the top of the window to follow the steps below in the OpenShift web console as part of this lab guide.

### 4.  Search for the Application Template

> **NOTE:** We recommended making the lab instructions pane in the lab environment larger by sliding the handle to the right. This will reduce the size of the terminal and in-lab web console but will better display the lab instructions and images.

If you are in the in the Administrator perspective, switch to Developer perspective and go to the `parksmap-< Your name >-namespace` project. 

<img width="264" alt="Screen Shot 2022-07-31 at 15 48 22 PM" src="https://user-images.githubusercontent.com/64369864/182114562-5ce00043-08c5-426f-b8e6-87b4ae55729f.png">

From the menu, select the `+Add` panel. Find the **parksmap-< Your name >-namespace** project and select it (if you're not asked to choose a project, it's probably because you've already selected one, simply go to the "**Project**" drop down at the top and select "**All Projects**" to continue)

![image](https://user-images.githubusercontent.com/64369864/182115364-cefa7a42-d547-4bc9-a2f4-7ce3ce26e1af.png)

You will see a screen where you have multiple options to deploy applications to OpenShift. Click `All Services` as shown below.

 <br/>
 
<img width="1204" alt="Screen Shot 2022-08-01 at 12 15 52" src="https://user-images.githubusercontent.com/64369864/182115868-f78e7837-83c7-4268-9a68-ce84bca47860.png">

 <br/>

We will be using `Templates` to deploy the application components. A template describes a set of objects that can be parameterised and processed to produce a list of objects for creation by OpenShift Container Platform. A template can be processed to create anything you have permission to create within a project, for example services, build configurations, and deployment configurations. A template can also define a set of labels to apply to every object defined in the template.

You can create a list of objects from a template using the CLI or, if a template has been uploaded to your project or the global template library, using the web console. In the `Search` text box, enter *parksmap* to find the application template that we've already pre-loaded for you

 <br/>

![image](https://user-images.githubusercontent.com/64369864/182122097-798e1e19-0825-47ff-bb67-d535ffca2a1c.png)

 <br/>

### 5. Instantiate the Application Template

Then click on the `Parksmap` template to open the popup menu and then click on the `Instantiate Template` button as shown below.

 <br/>

![image](https://user-images.githubusercontent.com/64369864/182122449-473c2cf1-a145-4a8b-90a4-b3ce69465c1a.png)

 <br/>

This will open a dialog that will *allow* you to configure the template. This template allows you to configure the following parameters:

- Parksmap Web Application Name
- Mlbparks Application Name
- Mlbparks MongoDB Application Name
- Nationalparks Application Name
- Nationalparks MongoDB Application Name
 <br/>

![image](https://user-images.githubusercontent.com/64369864/182122848-92db1006-4666-48ef-a279-899cc37f46d5.png)

 <br/>

Next click the blue *Create* button **without changing default parameters**. You will be directed to the *Topology* page, where you should see the visualization for the `parksmap` deployment config in the `workshop` application. OpenShift now creates all the Kubernetes resources to deploy the application, including *Deployment*, *Service*, and *Route*.


### 6. Check the Application

These few steps are the only ones you need to run to all 3 application components of `parksmap` on OpenShift. It will take a little while for the the `parksmap` application deployment to complete. Each OpenShift node that is asked to run the images of applications has to pull (download) it, if the node does not already have it cached locally. You can check on the status of the image download and deployment in the *Pod* details page, or from the command line with the `oc get pods` command to check the readiness of pods or you can monitor it from the Developer Console.

Your screen will end up looking something like this
 <br/> 

![image](https://user-images.githubusercontent.com/64369864/182124541-10fbb52d-fc83-40fd-b286-8740d1b2c229.png)

 <br/>

This is the *Topology* page, where you should see the visualisation for the `parksmap` ,`nationalparks`  and `mlbparks` deployments in the `workshop` application.


### 7. Access the Application

If you click on the `parksmap` entry in the Topology view, you will see some information about that deployment. 

 <br/>

![image](https://user-images.githubusercontent.com/64369864/182124777-b93dfb5c-654f-44c4-8d08-e42a855c0777.png)

 <br/>

On the "**Resources**" tab, you will see that there is a single *Route* which allows external access to the `parksmap` application. While the *Services* panel provide internal abstraction and load balancing information within the OpenShift environment. The way that external clients are able to access applications running in OpenShift is through the OpenShift routing layer. And the data object behind that is a *Route*. Also note that there is a decorator icon on the `parksmap` visualisation now. If you click that, it will open the URL for your *Route* in a browser

![image](https://user-images.githubusercontent.com/64369864/182124980-51a288db-99ff-49c0-8f59-d9bfdee61e6f.png)

This application is now available at the URL shown in the Developer Perspective. Click the link and you will see the following:

 <br/>

![image](https://user-images.githubusercontent.com/64369864/182125266-69158593-dc64-438b-b6d8-41b28fa68254.png)

 <br/>

You can notice that `parksmap` application does not show any parks as we haven't deployed database servers for the backends yet. We'll do that in the next step.


