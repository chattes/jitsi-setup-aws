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

[Reference](https://github.com/jitsi/lib-jitsi-meet/blob/master/doc/tokens.md)

We will use JWT Token Based Auth to secure the server, as with above setup anyone having the link to our domain can start making Video Calls.

* Patch Prosody Plugin from [here](https://packages.prosody.im/debian/pool/main/p/prosody-trunk/)



> For Ubuntu 18.04 you can download prosody-trunk_1nightly747-1~trusty_amd64.deb

```
wget https://packages.prosody.im/debian/pool/main/p/prosody-trunk/prosody-trunk_1nightly747-1~trusty_amd64.deb

dpkg -i prosody-trunk_1nightly747-1~trusty_amd64.deb
```

##### Installation jitsi-meet-tokens

Once Prosody plugin has been patched run

```
apt-get install jitsi-meet-tokens

```

It will ask for a Application id and Application Secret. Enter them.

You can use 
`hexdump -n 16 -e '4/4 "%08X" 1 "\n"' /dev/urandom` to generate a Secret

> For Example for my project I entered the Following

```
APP ID: cheappr
APP SECRET: D9E901D80FD5CB2A243263DB55EC8EC1
```

##### Configuring Prosody after installation

Open `/etc/prosody/prosody.cfg.lua` and add 
`Include "conf.d/*.cfg.lua"`
at the end of the file if it is not there.

In the same file check if client to server encryption is not enforced. Otherwise token authentication won't work:

```
c2s_require_encryption=false

```

**Next we are going to change the host config.**

Open `/etc/prosody/conf.avail/YOUR_DOMAIN.cfg.lua`

+ Set the plugins path `plugin_paths = { "/usr/share/jitsi-meet/prosody-plugins/" }`
+ Also optionally set the global settings for key authorization. Both these options default to the '*' parameter which means accept any issuer or audience string in incoming tokens, we us this when generating the JWT Tokens
```
asap_accepted_issuers = { "cheappr", "cheappr_iss" }
asap_accepted_audiences = { "jitsi", "cheappr_aud" }
```
+ Under you domain config change authentication to "token" and provide application ID, secret and optionally token lifetime. These lines should already be set but we have to check them once.

```
VirtualHost "cheappr.io"
    authentication = "token";
    app_id = "cheappr";             -- application identifier
    app_secret = "example_app_secret";     -- application secret known only to your token
                                           -- generator and the plugin
    allow_empty_token = false;             -- tokens are verified only if they are supplied by the client
```

+ Enable room name token verification plugin in your MUC component config section:

```
Component "conference.jitmeet.example.com" "muc"
    modules_enabled = { "token_verification" }
```




