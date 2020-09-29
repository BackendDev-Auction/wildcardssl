# Wildcard-SSL Automation with EFS

![alt tag](https://cf-templates-1itkybct44c2t-us-east-1.s3.amazonaws.com/autoc-efs-arch.png)


### Getting started 

Befor we start here we disscuss about Certbot and AWS EFS  because with the help of certbot & EFS  we can generate and store the SSL Certificates so, Certbot is an easy-to-use automatic client that fetches and deploys SSL/TLS certificates for your webserver and here is official website https://certbot.eff.org/

Now comes to the AWS EFS part. It is a storage service offer by AWS so  Amazon Elastic File System (Amazon EFS) provides a simple, scalable, fully managed elastic NFS file system for use with AWS Cloud services and on-premises resources. It is built to scale on demand to petabytes without disrupting applications, growing and shrinking automatically as you add and remove files, eliminating the need to provision and manage capacity to accommodate growth.



**STEP 1:** First of all Provision a EFS server by using AWS EFS service.

**Goto: > Filesystem > create file system > then click on customize option > choose name > select dev-auctionmobility-vpc and its subnets > do next ...**

![alt tag](
https://cf-templates-1itkybct44c2t-us-east-1.s3.amazonaws.com/efs-dash.png)



**STEP 2:** After that create 2 EC2 server one is SSL generate server (Master server) and another is slave server which consume SSL certificate through EFS.

**STEP 3:**  For SSL generation Go to the Master server and Install certbot and EFS utils package by this command here as follows:- 

```  sudo wget https://dl.eff.org/certbot-auto ```

``` sudo chmod +x certbot-auto ```

``` sudo apt-get -y install binutils ```

``` git clone https://github.com/aws/efs-utils ```

``` cd efs-utils/ && ./build-deb.sh ```

``` cd efs-utils/ && sudo apt-get -y install ./build/amazon-efs-utils*deb ```

**STEP 4:** 
**Mount EFS server to specific folder** so create any folder here i create with the name of efs

``` mkdir efs ```

**Here you can see that attach mount id** 

![alt tag](https://cf-templates-1itkybct44c2t-us-east-1.s3.amazonaws.com/eefs.png)

``` sudo mount -t efs -o tls fs-e08be162:/ efs/ ```



**STEP 5:**  Generate wild card SSL certificate with Certbot by this command:- 

``` sudo ./certbot-auto --config-dir efs/  --agree-tos --manual-public-ip-logging-ok --server https://acme-v02.api.letsencrypt.org/directory -d *.auctionmobility.com --manual --preferred-challenges dns-01 certonly --email abc@gmail.com  ```

**Note :- Here i configure the certificate path in efs/ folder by this command --config-dir efs/**

**expected output:**

```
Plugins selected: Authenticator manual, Installer None
Obtaining a new certificate
Performing the following challenges:
dns-01 challenge for domain.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.domain.com with the following value:
SiPbTUGdqp37WnMNnG17N4qoZEVIiuO_MivrrhYmW-Y
Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

**After that create TXT record in your Route53 with its value as above and then press enter for continue**

**Here you can see the output**

```
Press Enter to Continue
Waiting for verification...
Cleaning up challenges
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/domain.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/domain.com/privkey.pem
   Your cert will expire on 2020-09-06. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"


```


**(Optional) For Renew certificate use this command**  

``` sudo ./certbot-auto --config-dir efs/  --agree-tos --manual-public-ip-logging-ok --server https://acme-v02.api.letsencrypt.org/directory -d *.auctionmobility.com --manual --preferred-challenges dns-01 certonly --force-renewal --email abc@gmail.com ```


**STEP 6:** After that go to the Slave server and run the bash script here as follows:- 

**Note** It consume SSL certificate through EFS. (You can also pass this script into user-data while launching the instance)

``` 
#! /bin/bash
sudo apt-get update -y
sudo apt-get -y install binutils
git clone https://github.com/aws/efs-utils /home/ubuntu/efs-utils
cd /home/ubuntu/efs-utils/ && ./build-deb.sh
cd /home/ubuntu/efs-utils/ && sudo apt-get -y install ./build/amazon-efs-utils*deb
mkdir -p /etc/auctionmobility
sudo mount -t efs -o tls fs-e08be162:/ /etc/auctionmobility

```
