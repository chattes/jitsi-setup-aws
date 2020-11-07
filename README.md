# jitsi-setup-aws
Jitsi Meet Server Setup with JWT Authentication

## Jitsi Server Setup in AWS

## Prerequisite for Setup

* We will be setting up in Ubuntu 18.04
* We need a FQDN and an Elastic IP associated with the EC2 Instance
* The Security Group should have the following ports open

```
22/tcp
80/tcp
443/tcp
4443/tcp
5222/tcp
5347/tcp
10000/udp
3478/tcp
5349/udp
```

#### Setup

First of change to root user

```
sudo su 

```

> Goto /etc/systemd/system.conf

Add the following values if not there

```
DefaultLimitNOFILE=65000
DefaultLimitNPROC=65000
DefaultTasksMax=65000

```

```
apt update

apt install apt-transport-https

apt-add-repository universe

```

Setup FQDN (Example: cheappr.io)

```
sudo hostnamectl set-hostname meet

# Open the hosts file
vim /etc/hosts

127.0.0.1 localhost
<Your Servers Public IP> <Domain Name> meet
```

#### Add Jitsi Public Repository

```
curl https://download.jitsi.org/jitsi-key.gpg.key | sudo sh -c 'gpg --dearmor > /usr/share/keyrings/jitsi-keyring.gpg'
echo 'deb [signed-by=/usr/share/keyrings/jitsi-keyring.gpg] https://download.jitsi.org stable/' | sudo tee /etc/apt/sources.list.d/jitsi-stable.list > /dev/null

apt update
```

#### Install jitsi-meet

During Installatio a pop up comes up.
> The hostname of the current installation

Set your domain name here:

` I setup cheappr.io for Example `

We also need to setup a SSL Cert for HTTPS to work. Webrtc in Browser does not work very well without HTTPS.

Just select `Generate a new self-signed certificate`

Later we will run a script to generate the certs using LetsEncrypt

```
apt install jitsi-meet

```

#### Generate LetsEncrypt Certs

```
/usr/share/jitsi-meet/scripts/install-letsencrypt-cert.sh
```

At this point your basic Jitsi installation is Done!
Just hit your domain Eg: `https://cheappr.io` and you should get a nice page to start your conference.


#### Securing the Server with Prosody Configuration

We will use JWT Token Based Auth to secure the server, as with above setup anyone having the link to our domain can start making Video Calls.




