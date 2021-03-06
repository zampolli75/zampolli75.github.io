---
title: "Run Node apps with PM2"
layout: post
date: 2018-01-10 08:00
image: /assets/images/markdown.jpg
headerImage: false
projects: false
hidden: false # don't count this post in blog pagination
tag:
- node
- pm2
star: false
category: blog
author: jrod
description: Run Node applications with PM2 process manager in multiple environments
externalLink: false
---

# What this post is about
Being a rookie with process management guides, I was surprised when the Node application that I was running on a remote server will shut down every time I closed the SSH connection on the terminal. In fact, I soon discovered that running processes on the terminal are shut down when the SSH connection is closed.  
So to keep my application running at all times I needed a way to keep the process running on the background, and that is when I came across [PM2](http://pm2.keymetrics.io), a process manager for Node.  
Another strength of PM2 is that it is capable of managing multiple environments or applications using configuration files.  

In this post I will cover:  
- How to install PM2 and run applications with it  
- How to deploy the same application in multiple environments on the same machine  

# Install PM2
As PM2 is itself a Node package, its installation can be done through npm (the package manager for javascript). We will use PM2 across applications, and therefore a global installation of the library is preferred. To do so, we prompt the following command in the terminal:  

{% highlight shell %}
$ npm install pm2 -g |
{% endhighlight %}

We can confirm that the installation of PM2 was successful by quering the version of PM2.  

{% highlight shell %}
$ pm2 --version
{% endhighlight %}

# Running an application with PM2
To run a Node application with PM2 we can use the ```start``` command and pass as an argument the filename of the application script. For instance: 

{% highlight shell %}
$ sudo pm2 start app.js
{% endhighlight %}

Some applications run on ports that require admin rights. Therefore, if the application does not start you might want to try to run it again using ```sudo```. This is necessary for my application as it runs on port 80, which requires admin rights.  

After starting the application, PM2 forks the application folder and runs the process in the background. This allows making changes to the application files without interfering with the running process. Furthermore, when starting the application on a remote machine this allows keeping the process also running when the SSH connection is closed.  

# Running multiple environments
PM2 allows managing multiple environments through a configuration file. However, although this feature works for most use cases, it was coming it had a shortcoming. In fact, the definition of multiple environments assumes that you run them on different machines. On the contrary, we want to run the testing and production environments on the same machine. Furthermore, we want to have a demo environment of our app on the same machine.  
Therefore, we opted for an approach where we treated the different environments as different PM2 applications. To do so, we configure the ```ecosystem.config.js``` file with all the environments we want to create, treating them as different applications. For each of the applications, we define specific variables.

{% highlight javascript %}
module.exports = {
  apps : [{
    name        : "isds1102-test",
    script      : "./app.js",
    env: {
      "NODE_ENV": "test",
      "PORThttp": "4000",
      "URLdatabase": "mongodb://root:password@127.0.0.1:27017/dbname?authSource=admin",
      "URLauthFB": "http://ddslab.org:4000/auth/facebook/callback",
      "activationEmail": "email@lsu.edu",
    }
  },{
    name       : "isds1102-production",
    script     : "./app.js",
    env: {
      "NODE_ENV": "production",
      "PORThttp": "80",
      "PORThttps": "443",
      "URLdatabase": "mongodb://root:password@127.0.0.1:27017/dbname?authSource=admin",
      "URLauthFB": "http://ddslab.org/auth/facebook/callback",
      "activationEmail": "email@lsu.edu",
    }
  }]
}
{% endhighlight %}

Organizing our environments this way, we need to define the name of the environment that we want to run when starting the application. To do so, we have passed the name of the environment as an argument of PM2 when starting the application. Furthermore, we also have to pass as an argument the name of the file that contains the environment configurations.  

{% highlight shell %}
$ sudo pm2 start ecosystem.config.js --only isds1102-production
{% endhighlight %}

The above argument ```--only``` tells PM2 to only start the defined application name among all the ones contained in the ```ecosystem.config.js``` file.

# Handy commands to run my application
Following I document some of the most common commands I use to run my application.  

## Start environments
These are the different commands to start the multiple environments of the application:

{% highlight shell %}
<!-- production environment -->
$ sudo pm2 start ecosystem.config.js --only isds1102-production

<!-- test environment -->
$ sudo pm2 start ecosystem.config.js --only isds1102-test

<!-- development enviroment -->
$ sudo pm2 start ecosystem.config.js --only isds1102-development
{% endhighlight %}

The ```development``` environment is meant to be run only locally as it uses the ```localhost``` as address.

## Common PM2 commands
These are some of the common commands I use with PM2:

{% highlight shell %}
<!-- restart an application -->
$ sudo pm2 restart ecosystem.config.js --only isds1102-production

<!-- list all the running applications -->
$ sudo pm2 ls

<!-- stop an application -->
$ sudo pm2 stop isds1102-development

<!-- delete an application -->
$ sudo pm2 delete isds1102-development
{% endhighlight %}

For more information about PM2 consult its [documentation](http://pm2.keymetrics.io/docs/usage/cluster-mode/).
