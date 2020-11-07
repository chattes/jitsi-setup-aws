# jitsi-setup-aws
Jitsi Meet Server Setup with JWT Authentication

## Jitsi Server Setup in AWS

### Prerequisite for Setup

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

