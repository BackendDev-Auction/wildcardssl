# Wildcard-SSL Automation with EFS

![alt tag](https://cf-templates-1itkybct44c2t-us-east-1.s3.amazonaws.com/autoc-efs-arch.png)


### Getting started 

**STEP 1:** First of all Provision a EFS server by using AWS EFS. 


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


**(Optional)** For Renew certificate use this command**  

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
