---
layout: post
comments: true
title: Set Up JupyterHub on EC2
---

Having been using Jupyter Notebook to run my Python code almost exlusively for quite some time, I started to realize that it'd great if, instead of hosting Jupyter on my poor laptop, there is a server that does the hosting. I could for example, kick off a machine learning code before I leave work, and check the results when I get home. Oops I absolutely meant when I get work the next morning; who works extra hours anyway?

Some quick googling told me not only can you host a single-user Jupyter Notebook, there is also an multi-user option called [JupyterHub](https://github.com/jupyterhub/jupyterhub). I immediately decided to build one JupyterHub server on Amazon EC2 for our data science team, which potentially would be great for:

+ having a unified data science environment (e.g., Anaconda)
+ presenting code and results to colleagues
+ accessing S3 (where we store our data) much faster
+ prototyping Spark code before submitting it to EMR (with [Apache Toree](https://toree.apache.org))

# What you need
+ An Ubuntu EC2 instance; I ended up launching Ubuntu 14.04 because 16.04 didn't work well with JupyterHub for me.

# Installation steps
1. ssh to your newly launched EC2 instance
2. Install Anaconda for Python 2, which will be the main data science environment

```bash
wget https://repo.continuum.io/archive/Anaconda2-4.2.0-Linux-x86_64.sh
sudo bash Anaconda2-4.2.0-Linux-x86_64.sh 
```
When prompted, you might want to change the Anaconda installation prefix to /anaconda2.

3. Install necessary libraries for JupyterHub (Note: JupyterHub runs on Python 3, make sure you differentiate it from the Anaconda Python 2 installed in step 1)

```bash
sudo apt-get update
sudo apt-get install python3-pip 
sudo apt-get install npm nodejs-legacy 
sudo npm install -g configurable-http-proxy 
sudo pip3 install jupyterhub 
sudo pip3 install --upgrade notebook 
```
4. Create an SSL key and a certificate for the HTTPS connection to the server.

```bash
openssl genrsa 1024 > host.key
chmod 400 host.key
openssl req -new -x509 -nodes -sha1 -days 365 -key host.key -out host.cert
```
5. Configure JupyterHub

```bash
# generate the config file
jupyterhub --generate-config 
```
This generates a file named __jupyterhub_config.py__; edit this file by appending the following to the end:

```
c.LocalAuthenticator.create_system_users = True
#c.Authenticator.whitelist = set()
c.Authenticator.admin_users = {'rex'}
c.Spawner.notebook_dir = '~/notebooks'
c.JupyterHub.ssl_cert = 'host.cert'
c.JupyterHub.ssl_key = 'host.key'
c.JupyterHub.port = 443
```

6. Create new users

JupyterHub users are essentially Linux users who log in JupyterHub with their Linux credentials. New accounts can be set up by running:

```bash
# create a new user; you need to fill in the password and so on
sudo adduser rex
su rex
# cd to the home directory and make a directory for JupyterHub
cd
mkdir notebooks
exit
	```

7. Add Anaconda Python 2 as a Jupyter kernel

```bash
# check existing kernels
sudo jupyter kernelspec list 
```
This should return:

```
Available kernels:
  python3    /usr/local/lib/python3.4/dist-packages/ipykernel/resources
```
	
Add Anaconda Python 2 by

```bash
# /anaconda2/bin/python2 is not in the PATH of sudo; use its absolute path
sudo /anaconda2/bin/python2 -m ipykernel install 
```

```bash
# check existing kernels again
sudo jupyter kernelspec list 
```
This should now return:

```
Available kernels:
  python3    /usr/local/lib/python3.4/dist-packages/ipykernel/resources
  python2    /usr/local/share/jupyter/kernels/python2
```
	
8. Launch JupyterHub

```bash
sudo jupyterhub
```
Enjoy!
	
