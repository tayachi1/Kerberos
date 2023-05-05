# Kerberos authentification with PostgreSQL

This project is a step by step of the configuration of Kerberos authentication for PostgreSQL.

## WHAT IS KERBEROS:
Kerberos is a network authentication protocol designed to provide secure authentication for client/server applications by using secret-key cryptography. It was developed by the Massachusetts Institute of Technology (MIT) in the 1980s and has since become a widely-used protocol in enterprise environments.
## ARCHITECTURE:
We have three instances : kerberos server , PostgreSQL server and the client 
## SOME KERBEROS TERMINOLOGY
- Realm: A realm in Kerberos is a logical grouping of networked computers, services, and users that share a common authentication database, identified by a unique realm name.
- Principal: a principal is an entity (a user or a service) that is identified by a principal name, consisting of a primary component and a realm. Each principal is issued a secret key that is used to authenticate them when accessing network resources.
- KDC: KDC stands for Key Distribution Center, which is a trusted third-party authentication server that issues and manages authentication tickets and secret keys used by principals to access network services.
- TGT: TGT stands for Ticket-Granting Ticket, which is a type of ticket issued by the Key Distribution Center (KDC) to a principal after successful authentication. The TGT can be used by the principal to obtain other tickets, including service tickets, without the need for further authentication. The TGT is encrypted with the principal's secret key and includes a time stamp and other information that is used to enforce ticket expiration and limit ticket reuse.
- Keytab: a keytab is a file that contains one or more secret keys for one or more principals. The keytab is typically used by services to authenticate with the Key Distribution Center (KDC) and obtain service tickets without requiring interactive user input. The keytab file is encrypted with a master key that is known only to the KDC and the keytab owner, and is usually stored on the file system of the service's host machine.

- Client : a workstation that needs to access a Service in the kerberos realm.

## STEPS
To follow along with this demonstration, we'll need to perform the following steps:

- Disable the firewall on all instances by running the command 
```bash 
systemctl stop firewalld 
```
on each instance.

- Check the IP addresses of all instances and improve readability by editing the /etc/hosts file (you can use the ifconfig command on the command line to check your IP address on the network).

- Configure the KDC server by installing the krb5 server and client with the command yum install krb5-server krb5-workstation pan_krb5.

- Edit the file /var/kerberos/krd5kdc/kdc.conf and ensure that it resembles the sample configuration provided in the demo.

- Edit the file /etc/krb5.conf to match the sample configuration provided in the demo.

- Edit the file /var/kerberos/krb5kdc/kadm5.acl to include */admin@RT.COM so that the admin user can connect with default privileges.

- Create a Kerberos database by running the command 
```bash 
kdb5_util create -s -r RT.COM 
```
and entering the database password when prompted.

- Start the services krb5kdc and kadmin with the command 
```bash 
start krb5kdc kadmin
```

- Enter the KDC as the root user by running the command 
 ```bash 
 kadmin.local
 ```

- Add the necessary principals with their passwords, including root/admin, ghassen (which will be our client for PostgreSQL), and postgres/server.rt.com (a special service supported by GSSAPI).

- Extract the keytab file by exiting from kadmin.local, running the kutil command, and running the add_entry command to extract the key for the postgres/server.rt.com service.

- Write the extracted key to the keytab file by running the wkt postgres.keytab command (where postgres is the name of the keytab).

- Copy the keytab file to the PostgreSQL server using the scp command
```bash
sudo scp postgres.keytab rt@192.168.56.101:/home/rt/keytab
```
- On the PostgreSQL server, edit the postgres.conf file to uncomment the krb_server_keyfile line and link it to the keytab that was transferred from the KDC server. Additionally, ensure that the listen_addresses setting is set to * and the port is set to 5444.

- Edit the pg_hba.conf file to ensure that it allows ghassen client to access the database from outside this instance.

- Install the Kerberos client on the client server by running the command 
```bash 
yum install krb5-workstation pam_krb5 -y
```

- Make sure that the krb5.conf file on the client server matches the configuration provided in the KDC server.

- Obtain a ticket from the KDC by running the command 
```bash
kinit ghassen
```

- Finally, connect to the PostgreSQL server by running the command 
```bash 
psql -h server.rt.com -U ghassen
```
