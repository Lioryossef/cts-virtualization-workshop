## Create a User for the PowerDNS Server

Allowing access to our database remotely is important due to it being a separate instance from our PowerDNS. Creating a database user and allowing it access.
  1. Create a user by the following command.

`create user <’username’> identified by <’password’>;` 

  2. Permit the user that was created to access the instance remotely.

```
grant all privileges on powerdns.* to <’username’>@’%’ identified by <'password'>;

flush privileges;
```

<img width="678" alt="Screen Shot 2022-12-26 at 11 00 09" src="https://user-images.githubusercontent.com/64369864/209528763-6eab80ad-489a-4187-8e8d-54de3e782103.png">

<img width="327" alt="Screen Shot 2022-12-26 at 11 00 32" src="https://user-images.githubusercontent.com/64369864/209528804-d96c1a1b-53e3-4173-bbb3-4208ede41eab.png">

## Install and Configure the PowerDNS

  1. Connect to the PowerDNS instance and Install the actual PowerDNS packages.

```
dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
yum-config-manager --disable epel
dnf install --enablerepo=epel -y pdns pdns-backend-mysql
```

  2. Copy the structure of the tables for the PowerDNS database

`cat /usr/share/doc/pdns/schema.mysql.sql` 

<img width="685" alt="Screen Shot 2022-12-26 at 11 02 34" src="https://user-images.githubusercontent.com/64369864/209529015-1395fbe8-7319-4619-8757-266024a49ac0.png">

  3. Connect to MariaDB from the first instance (mariadb-0).

`mysql -u root -p`

  4. Create in mariadb-0 instance the structure of the tables for the PowerDNS database by running the following MySQL queries below.

`use powerdns;`

  5. Paste the schema from the ‘/usr/share/doc/pdns/schema.mysql.sql’ file.

  6. Create a label and one service for all mariadb instances

  7. Create a label and allocate to each mariadb instances for example mariadb1

  <img width="693" alt="Screen Shot 2022-12-26 at 11 04 27" src="https://user-images.githubusercontent.com/64369864/209529233-b6383b19-9078-4cf4-b88a-40a201c5dc55.png">

<img width="687" alt="Screen Shot 2022-12-26 at 11 05 04" src="https://user-images.githubusercontent.com/64369864/209529287-b814a50e-5040-46bb-930e-3dca0b403e62.png">

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
    - protocol: TCP
      name: tcp-3306
      port: 3306
      targetPort: 3306
    - protocol: TCP
      name: tcp-4567
      port: 4567
      targetPort: 4567
    - protocol: TCP
      name: tcp-4568
      port: 4568
      targetPort: 4568
    - protocol: TCP
      name: tcp-4444
      port: 4444
      targetPort: 4444
    - protocol: UDP
      name: udp-4567
      port: 4567
      targetPort: 4567
```

<img width="700" alt="Screen Shot 2022-12-26 at 11 06 04" src="https://user-images.githubusercontent.com/64369864/209529384-5eb9cd71-f2d2-4770-a881-7094eb0b570f.png">

  8. Return to the powerdns-0 instance and configure the instance.

`sudo vi /etc/pdns/pdns.conf`

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

  9. Start pdns service 

`sudo systemctl start pdns`

  10. Create a DNS zone and record.

```
sudo pdnsutil create-zone <domain_name>
sudo pdnsutil add-record <domain_name> <record> A <ip_address>
```

<img width="674" alt="Screen Shot 2022-12-26 at 11 08 50" src="https://user-images.githubusercontent.com/64369864/209529708-23678ca5-966e-498d-b06e-8435ad6c0c82.png">

  11. Once the DNS zone is created the record will be stored in our database. In this step check the record data on the first instance (mariadb-0).
```
mysql -u root -p

use powerdns;

select  * from domains; 

```
<img width="685" alt="Screen Shot 2022-12-26 at 11 10 03" src="https://user-images.githubusercontent.com/64369864/209529844-1e9bf00d-56b9-49c6-b023-d9dff06af62a.png">

```
select * from records;
```
<img width="704" alt="Screen Shot 2022-12-26 at 11 10 50" src="https://user-images.githubusercontent.com/64369864/209529917-d40530e4-e3b9-4c86-b14d-1e5d5940f277.png">

  12. Repeat the command to check the record data on the second instance (mariadb-1). This step shows that the data has indeed been replicated and thus any data that will be created on one of the instances will replicate to the other one

<img width="708" alt="Screen Shot 2022-12-26 at 11 11 32" src="https://user-images.githubusercontent.com/64369864/209530008-90004f8a-e30f-432c-bc3d-32fa7a200ef0.png">

<img width="707" alt="Screen Shot 2022-12-26 at 11 11 45" src="https://user-images.githubusercontent.com/64369864/209530029-2ef9250d-e3d0-4493-97cc-bf8cea811731.png">

