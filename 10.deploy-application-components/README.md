In this lab, we will use the OpenShift Web Console to deploy the frontend and backend components, which comprises of one frontend web application, and 2 databases:

### 1. Creating the Project

As a first step, we need to create a project where our application will be deployed. You can create the project with the following command:

```execute
oc new-project workshop-< Your name >-namespace
```

### 2. Navigate to the OpenShift Web Console

Select the blue "**Console**" button at the top of the window to follow the steps below in the OpenShift web console as part of this lab guide.

### 3. Create VMs from Openshift console
- Click Virtualization → Create Virtual Machine.
- Choose a template, In this case, Red Hat Enterprise Linux 8.0+ was chosen.

<img width="704" alt="Screen Shot 2022-11-10 at 14 35 24" src="https://user-images.githubusercontent.com/64369864/201093126-6de8d0c2-5649-4933-b4d4-0e6d67f39737.png">

- Provide a custom boot source

<img width="680" alt="Screen Shot 2022-11-10 at 14 36 26" src="https://user-images.githubusercontent.com/64369864/201093317-fa7f8aff-8b80-4683-9aa1-68e21e62f618.png">

<img width="694" alt="Screen Shot 2022-11-10 at 14 38 10" src="https://user-images.githubusercontent.com/64369864/201093624-5cd2b95c-9911-48c6-81eb-56d366279c3c.png">
