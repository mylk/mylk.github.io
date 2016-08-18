---
layout: post
title: PHPUnit Selenium tests without desktop environment
---

You feel proud you test your software.
You add features (and even refactor!) and you know there will be no surprises on production.
In case anything breaks, you will be the first to know because your tests will tell you about it.

I like to call this, "mental health".

## Why you should do this?

### Scenario 1
Have you noticed that your PHPUnit Selenium tests are really disturbing when the browser pops-up interrupting you from your job?  
Your tests may run for 15 minutes and the browser pops-up for each test case.  
If you have twenty of them, it gets nerve wracking.

### Scenario 2
Your CI has to run your PHPUnit Selenium tests.
The host that will run the tests has to have a desktop environment and a browser installed.
That's too much for your cloud hosted server and it doesn't sound nice in general.

You may say you could have an extra box with a desktop environment to run the browser. Your own development computer or any cheap box in your office.
In this case you have to setup remote port forwarding so the host that runs PHPUnit forwards the traffic to the box running Selenium server and the browser.  

You also have to take care to keep the persistent connection of the port forwarding alive. Hm, sounds messy to me.

## The solution
The server that runs your tests can also run the Selenium server and the browser, without having a desktop environment
or do remote port forwarding.

Using the virtual display server [Xvfb](https://en.wikipedia.org/wiki/Xvfb), you can perform
graphical operations in memory without showing any output. Let's say you fool applications that require a display server
to behave like there is one.

A tested and working solution consists of the following software:

- Latest stable Xvfb
- Selenium standalone server v2.45.0
- Chromedriver v2.22
- Latest stable Google Chrome browser

## Downloading the software
Install Xvfb:

```
sudo apt-get install xvfb
```
If you run Debian or Ubuntu, the command above is for you. Else, use the package manager of your distribution.

Lets download Selenium server and Chromedriver.  
I prefer to run my tests on a Chrome browser. If you prefer another (or additional) browsers, download the corresponding driver(s).

We will store Selenium server and Chromedriver in "/var/selenium/":

```
sudo mkdir /var/selenium/
```

Download Selenium server:

```
wget http://selenium-release.storage.googleapis.com/2.45/selenium-server-standalone-2.45.0.jar

sudo chmod +x selenium-server-standalone-2.45.0.jar

sudo mv selenium-server-standalone-2.45.0.jar /var/selenium/
```

Download Chromedriver:

```
wget http://chromedriver.storage.googleapis.com/2.22/chromedriver_linux64.zip

unzip chromedriver_linux64.zip

sudo chmod +x chromedriver

sudo mv chromedriver /var/selenium/
```

Install the latest stable Google Chrome browser:

```
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb

sudo dpkg -i google-chrome-*.deb
```

"deb" packages and the "dpkg" application are only available to Debian based distros. If you don't run one of them,
use your distro's packages and package manager.

## Get all things running
You are already able to run all that software manually, but lets make it look like a more permanent solution
by creating start-up scripts for Xvfb and Selenium server.

That way, you are able to run all software components manually, but also they will automatically run in case
your system reboots.

If you don't run the latest version of your GNU/Linux distro, you may still use ```init.d``` as your init system.
In case you follow the changes of the GNU/Linux ecosystem, you may be using ```systemd```.

> An init system is a process that handles initializing and managing system services and daemons
when your system boots and runs.

### In case init.d is used:

Put the content of the following into ```/etc/init.d/xvfb```:

```
https://gist.github.com/mylk/f220499c16851b241d1098d21cce0eec
```

And the content of the following into ```/etc/init.d/selenium```:

```
https://gist.github.com/mylk/edc052c329643e526d0eee695ab50afa
```

Make the scripts executable:

```
sudo chmod +x /etc/init.d/xvfb
sudo chmod +x /etc/init.d/selenium
```

Enable the scripts to run in system's start-up:

```
sudo update-rc.d xvfb defaults
sudo update-rc.d selenium defaults
```

Start the services:

```
sudo /etc/init.d/xvfb start
sudo /etc/init.d/selenium start
```

### In case systemd is used:

Put the content of the following into ```/etc/systemd/system/xvfb.service```:

```
https://gist.github.com/mylk/709a68a7e7d635b062280af36127d9d1
```

And the content of the following into ```/etc/systemd/system/selenium.service```

```
https://gist.github.com/mylk/6fe812863c5a44f7ff8116f48cc22492
```

Make the scripts executable:

```
sudo chmod +x /etc/systemd/system/xvfb.service
sudo chmod +x /etc/systemd/system/selenium.service
```

Enable the scripts to run in system's start-up:

```
sudo systemctl enable xvfb.service
sudo systemctl enable selenium.service
```

Starting Selenium should automatically start Xvfb too, as its defined as a dependency:

```
sudo systemctl start selenium.service
```

Start your Selenium tests as usual and they will be running on your virtual display server!

## Conclusion
Often, the cleanest and more permanent solution is the one that consists of the less components
and the one that needs the less human intervention.

In the described solution we use the minimum number of hosts and setup / environment prerequisites.

We are also prepared for scenarios like system failures and reboots. The less actions you have to do in those cases,
the better for you and your fellow sysadmins.

Happy testing!
