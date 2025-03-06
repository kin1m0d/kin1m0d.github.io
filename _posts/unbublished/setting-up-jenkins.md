---
layout: post
title:  "Jenkins Setup"
date:   2025-02-?? 12:59:35 +0000
categories: jekyll update
---

# Day 1

Undust the raspi, ssh to it and install Jenkins. Well, there is no connection... why not?

Troubleshooting ...

I have plugged in an ethernet cable, which I would have done anyways, but still, I wanted to make the Wi-Fi work, annoying -.-

# Day 2




sudo apt install openjdk-11-jre


pi@raspi:~ $ sudo apt install openjdk-11-jre
Reading package lists... Done
Building dependency tree
Reading state information... Done
openjdk-11-jre is already the newest version (11.0.23+9-1~deb10u1).
openjdk-11-jre set to manually installed.
The following package was automatically installed and is no longer required:
  wmdocker
Use 'sudo apt autoremove' to remove it.
0 upgraded, 0 newly installed, 0 to remove and 2 not upgraded.


https://www.jenkins.io/doc/book/installing/linux/



```bash
pi@raspi:~ $ sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
>   https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
--2025-02-19 22:16:22--  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
Resolving pkg.jenkins.io (pkg.jenkins.io)... 151.101.190.133, 2a04:4e42:82::645
Connecting to pkg.jenkins.io (pkg.jenkins.io)|151.101.190.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3175 (3.1K) [application/pgp-keys]
Saving to: ‘/usr/share/keyrings/jenkins-keyring.asc’

/usr/share/keyrings/jenkins- 100%[==============================================>]   3.10K  --.-KB/s    in 0s

2025-02-19 22:16:24 (7.63 MB/s) - ‘/usr/share/keyrings/jenkins-keyring.asc’ saved [3175/3175]
```


```bash
pi@raspi:~ $ echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
>   https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
>   /etc/apt/sources.list.d/jenkins.list > /dev/null
```




```bash
pi@raspi:~ $ sudo apt update
Hit:1 http://raspbian.raspberrypi.org/raspbian buster InRelease
Ign:2 https://pkg.jenkins.io/debian-stable binary/ InRelease
Hit:3 http://archive.raspberrypi.org/debian buster InRelease
Get:4 https://pkg.jenkins.io/debian-stable binary/ Release [2,044 B]
Get:5 https://pkg.jenkins.io/debian-stable binary/ Release.gpg [833 B]
Get:6 https://download.docker.com/linux/raspbian buster InRelease [33.6 kB]
Get:7 https://pkg.jenkins.io/debian-stable binary/ Packages [28.5 kB]
Fetched 65.0 kB in 2s (36.0 kB/s)
Reading package lists... Done
Building dependency tree
Reading state information... Done
2 packages can be upgraded. Run 'apt list --upgradable' to see them.
```





sudo apt install jenkins

installs jenkins and

```
The package installation will:

Setup Jenkins as a daemon launched on start. Run systemctl cat jenkins for more details.

Create a ‘jenkins’ user to run this service.

Direct console log output to systemd-journald. Run journalctl -u jenkins.service if you are troubleshooting Jenkins.

Populate /lib/systemd/system/jenkins.service with configuration parameters for the launch, e.g JENKINS_HOME

Set Jenkins to listen on port 8080. Access this port with your browser to start configuration.
```





journalctl -xe
shows me that the java version is wrong
```log
Feb 19 22:27:44 raspi jenkins[3113]: Running with Java 11 from /usr/lib/jvm/java-11-openjdk-armhf, which is older th
Feb 19 22:27:44 raspi jenkins[3113]: Supported Java versions are: [17, 21]
Feb 19 22:27:44 raspi jenkins[3113]: See https://jenkins.io/redirect/java-support/ for more information.
Feb 19 22:27:44 raspi systemd[1]: jenkins.service: Main process exited, code=exited, status=1/FAILURE
```

And yeah.. it's pointing to 11 :smiling_face_with_tear:
```
pi@raspi:~ $ java --version
openjdk 11.0.23 2024-04-16
OpenJDK Runtime Environment (build 11.0.23+9-post-Raspbian-1deb10u1)
OpenJDK Server VM (build 11.0.23+9-post-Raspbian-1deb10u1, mixed mode)
```

Of course it doesn't work...
```
pi@raspi:~ $ sudo apt install openjdk-17-jre
Reading package lists... Done
Building dependency tree
Reading state information... Done
E: Unable to locate package openjdk-17-jre
```



Great
> Raspbian 10 only has OpenJDK 11; to install the packaged version of OpenJDK 17 on Raspbian, you should upgrade to Raspbian 11.
https://unix.stackexchange.com/questions/762905/install-jdk17-on-raspberrypi


Now I need upgrade, or maybe find a better OS for my pi?


Also I noticed a low voltage warning when I was connected via VSC, even though I'm using the offical power supply?