## Ensure the DNS record 

  1. Create VM for test DNS record 

<img width="722" alt="Screen Shot 2022-12-26 at 11 32 11" src="https://user-images.githubusercontent.com/64369864/209532315-1f605611-3283-48f8-8bc7-f41389ce4adf.png">

<img width="723" alt="Screen Shot 2022-12-26 at 11 32 41" src="https://user-images.githubusercontent.com/64369864/209532363-5f198a1d-a469-458f-9a97-c6d11e7d924c.png">

  2. Connect to the workstation VM install the actual bind-utils packages.

`sudo dnf install bind-utils`

  3. Check the DNS record. 

`dig <record>.<zone> @<powerdns IP>`

<img width="649" alt="Screen Shot 2022-12-26 at 11 34 17" src="https://user-images.githubusercontent.com/64369864/209532537-ad9fea03-1b1f-4187-bf65-caac8f8c5250.png">

## What is the Value of High-availability?

In this section of my blog I want us to understand the importance and value that the high-availability we have created gives us. As we saw at DB steps the data replicated and synchronized, the PowerDNS instance configured works with the second MariaDB instance(mariadb-1). Now first I check the DNS record and then I will create another DNS instance that will work with the first MariaDB instance(mariadb-0). Finally I will stop one of the mariadb instances to simulate a failure and we will see that our service remains available and prevents downtime to the customer.

  1. Create one more PowerDNS instance (powerdns-1). 

<img width="711" alt="Screen Shot 2022-12-26 at 11 36 04" src="https://user-images.githubusercontent.com/64369864/209532739-d42b547e-d556-4e5f-b017-33d3d60d9b67.png">

<img width="717" alt="Screen Shot 2022-12-26 at 11 36 27" src="https://user-images.githubusercontent.com/64369864/209532770-9b284f62-bc30-471d-a3a0-95baee442f65.png">

  2. Create label and allocate for each PowerDNS instance.

<img width="590" alt="Screen Shot 2022-12-26 at 11 38 57" src="https://user-images.githubusercontent.com/64369864/209533025-3a3d73ea-a42c-4843-a4de-8818eaeb1a22.png">

<img width="699" alt="Screen Shot 2022-12-26 at 11 39 31" src="https://user-images.githubusercontent.com/64369864/209533109-9c4421f1-261f-49b3-ad23-1af9fac6de5b.png">

<img width="693" alt="Screen Shot 2022-12-26 at 11 41 08" src="https://user-images.githubusercontent.com/64369864/209533280-3bc30a2f-b36a-4b2b-bccf-515debda514d.png">

  3. Create a DNS service for PowerDNS instances
```
apiVersion: v1
kind: Service
metadata:
  name: <service name>
  namespace: <namespace>
spec:
  selector:
    < label >
  ports:
    - protocol: UDP
      name: udp-53
      port: 53
      targetPort: 53
```
<img width="600" alt="Screen Shot 2022-12-26 at 11 41 55" src="https://user-images.githubusercontent.com/64369864/209533374-2a746ea7-3d70-4335-abfd-ff5bf2bb52c8.png">

  4. Connect to the powerdns-1 instance and configure the instance.
```
sudo vi /etc/pdns/pdns.conf

```
```
launch=gmysql
gmysql-host=<mariadb-ports.galera-cluster.svc.cluster.local>
gmysql-dbname=powerdns
gmysql-user=<pdns>
gmysql-password=<qwe123>
api=yes
api-key=dbpass
webserver-address=127.0.0.1
logging-facility=0
loglevel=5
log-dns-queries=yes
resolver=[::1]:53
expand-alias=yes
```
### NOTE: If SElinux mode is enforced, create a SELinux policy that will allow that.

  4. Start pdns service.
`sudo systemctl start pdns`

  5. The DNS zone and record that I created before are automatically replicated and synced thanks to DB sync.

  6. Add the IP addresses of the DNS service from step 2 to the resolve.conf file of the workstation instance. The IPs will be overwritten by NetworkManage. The intention here is to show the HA for the DNS service in the next steps, another option is to use the dig common with PowerDNS service IP like: ‘dig <record>.<zone> @<powerdns IP>’ or with nmcli.

`vi /etc/resolv.conf` 

<img width="684" alt="Screen Shot 2022-12-26 at 11 44 52" src="https://user-images.githubusercontent.com/64369864/209533740-e3b4c753-29f7-4e5b-87c4-1fcd9610cc57.png">

  7. Check the DNS record.

<img width="612" alt="Screen Shot 2022-12-26 at 11 45 42" src="https://user-images.githubusercontent.com/64369864/209533824-b8965d4d-d9e3-432f-b845-48aae7d0e922.png">

  8. Stop the first instance (mariadb-0).
<img width="721" alt="Screen Shot 2022-12-26 at 11 46 12" src="https://user-images.githubusercontent.com/64369864/209533880-a1a49b2e-5b91-401f-9f4d-5f531a037582.png">

  9. Connect to the workstation instance and verify if the DNS record is still available. Pay attention to the server that answers.
<img width="631" alt="Screen Shot 2022-12-26 at 11 46 49" src="https://user-images.githubusercontent.com/64369864/209533960-c6d423eb-822f-480c-af87-6a8f0b74a7b6.png">

### NOTE: We saw that still had a connection to the mariadb-1instance and the DNS service remains available to the customer. That is, given that because I created a service that has a list backed for the DB instances and a service that has a list backed for the DNS instances the customer at the end gets one address and we are calm that our service remains available even if a DNS server or a DB server falls.

## Conclusion
As we have seen the data is synchronized and replicated between the members of the cluster. As a result, the DNS service will be available even if Galera cluster members fail. 
High availability is crucial when delivering a service to customers. It allows administrators to ensure continuous service availability, improve service mainability, as well as the ability to fix problems with no or minimal impact to customers.
OpenShift Virtualization allows you to run highly available VM workloads and ensures they will keep running even in a case of VM failures. 
This blog is part of a large and interesting project that allows us to set up a highly available DNS service on Kubernetes using the Galera cluster. This type of deployment benefits from Kubernetes cloud-native advantages and unified API that are used to simplify the configuration and management of containers and VMs on the same platform. 
