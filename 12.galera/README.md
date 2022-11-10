Access to the instances could be via SSH. This blog performs all actions using the personal workstation's direct access with SSH and not with OpenShift Web Console. Access to the VM remotely via SSH is important for both safety reasons (ssh being an encrypted protocol) and ease of use. The NodePort service could be configured via UI by enabling the "Expose SSH access to this virtual machine" in the VM creation wizard ( Starting from OpenShift Virtualization 4.8) or by yaml. So let's see how to expose VMs with the NodePort service.

- The UI option( OpenShift version > 4.8) by enabling the "Expose SSH access to this virtual machine" exists in the VM creation wizard. 

<img width="701" alt="Screen Shot 2022-11-10 at 15 00 29" src="https://user-images.githubusercontent.com/64369864/201098102-1cc948b6-8497-4356-927b-09c991a97a23.png">

- As an alternative way to create the NodePort, click Networking → Services and click Create Service.

<img width="673" alt="Screen Shot 2022-11-10 at 15 10 17" src="https://user-images.githubusercontent.com/64369864/201100202-bd3a5123-d67c-4cba-a38a-5a7543de778f.png">

- For example, a snapshot of yaml configuration that exposes the first VM in my project (mariadb-0). The service type is NodePort and the TCP port is 22. 
```
apiVersion: v1
kind: Service
metadata:
  name: <service name>
  namespace: <name spase>
spec:
  externalTrafficPolicy: Cluster
  ports:
    - port: 22
      protocol: TCP
  selector:
    kubevirt.io/domain: <VM name>
  type: NodePort

```
<img width="679" alt="Screen Shot 2022-11-10 at 15 11 28" src="https://user-images.githubusercontent.com/64369864/201100471-756c2391-e273-4cef-abdd-b7844909ff5f.png">

> Perform the same steps for each VM in the project.

Click on the service → Service Details page that shows the port assigned to the service, which in my example is 30116.

<img width="702" alt="Screen Shot 2022-11-10 at 15 12 24" src="https://user-images.githubusercontent.com/64369864/201100666-fe9e9a82-4f68-4194-9ada-bdcfc957a486.png">

For direct access, you would need to use the port and any cluster node IP address.
Click Compute → Nodes and click on the relevant worker to enter its Node Details page to see the IP address of the node. In my example, the IP is 10.20.0.202.

<img width="677" alt="Screen Shot 2022-11-10 at 15 13 15" src="https://user-images.githubusercontent.com/64369864/201100814-9eb866fb-5fc2-4fb4-85c7-5b47b8760027.png">

Open a SSH connection from the workstation to the VM using the information from the Node and Service details.

<img width="673" alt="Screen Shot 2022-11-10 at 15 13 42" src="https://user-images.githubusercontent.com/64369864/201100907-43904300-32f9-4d0d-94bb-64ad93f6b513.png">


