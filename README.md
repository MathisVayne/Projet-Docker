# Project-Docker

# Project Objectives

Creating a project consisting of a client, a server and a firewall that will form a large infrastructure. Each of these three parts will be in a Docker container. For this we were asked to work with Docker_compose.

# Installing Docker on a VM

It will first be necessary to update all the packages of the VM with the help of the apt update and apt upgrade commands

Then we will install the different necessary packages with sudo apt install which are: apt-transport- https / ca-certificates / curl / lsb-release

Retrieve the official Docker source and create a repository where you will add the Docker:
        curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
        
        echo \ "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
        $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/nul

Downloading Docker from the repository:
        apt install docker-ce 
        apt install docker-ce-cli
        apt install containerd.io
       
Now you have Docker and some supplies installed on your VM

# Installing docker-compose on your VM

The method will be the same as before with Docker, we will retrieve the url via Github:
        sudo curl -L https://github.com/docker/compose/releases/download/v2.0.1/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
        
This file will need a chane of permission and permit to run it:
        sudo chmod +x /usr/local/bin/docker-compose
       
Now that everything is installed, you will only have to write the different files that will make up your infrastructure   
        
# Contents of the different files    

First of all we will write a yml file that will allow us to create our 3 containers after its execution

  Docker.yml:
  
          version: "3"
          services:
          
             Server:
                build: Server/
                container_name: Server
                command: python ./server.py
                 ports:
                    - 985:985

            Client:
               build: Client/
               container_name: Client
               command: python ./client.py
               network_mode: host
               depends_on:
                  - Server

            Firewall:
                build: Firewall/
                container_name: Firewall
                sysctls:
                   - net.ipv4.ip_forward=1
                cap_add:
                   - NET_ADMIN
                command: >
                    bash -c "./clean.sh && ./fw.sh"
                depends_on:
                    - Client

For the Server part:
  
    server.py:
    
                !/usr/bin/env python3

                import http.server
                import socketserver
                handler = http.server.SimpleHTTPRequestHandler
                with socketserver.TCPServer(("", 985), handler) as httpd:
                httpd.server_forever()
  
 
  DockerFile (Server):
  
                FROM python:latest
                ADD server.py /Server/
                ADD Server.html /Server/
                WORKDIR /Server/
                
  Server.html:
              
              //You can write what you want on this web page
              
              Mathis Vayne's Web Server


For the Client Part:

   client.py:
   
              #!/usr/bin/env python3

              import urllib.request
              fp = urllib.request.urlopen("http://localhost:985/")
              encodedContent = fp.read()
              decodedContent = encodedContent.decode("utf8")
              print(decodedContent)
              fp.close()
              
  DockerFile (Client):
  
              FROM python:latest
              ADD client.py /Client/
              WORKDIR /Client/
              
Firewall part:

  clean.sh (this will earse all rules that were created for your firewall and we accept everything):
          
              #!/bin/bash

              iptables -F INPUT
              iptables -F OUTPUT
              iptables -F FORWARD

              iptables -P INPUT ACCEPT
              iptables -P OUTPUT ACCEPT
              iptables -P FORWARD ACCEPT
              
              iptables -L -v -n
  
  fw.sh(this will change and crete new rules for the firewall):
              
              iptables -P INPUT DROP
              iptables -P OUTPUT DROP
              iptables -P FORWARD DROP
              
              iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
              iptables -A OUTPUT -j ACCEPT
              iptables -A INPUT -p tcp --dport 443 -j ACCEPT
              iptables -A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
              iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
              
  DockerFile (Firewall):
              
              FROM python:latest
              ADD clean.sh /Firewall/
              ADD fw.sh /Firewall/
              WORKDIR /Firewall/
              
It is important to place these different files in the right place

# Organizing files

After writting all these files, you will have to organize them into a folder name ProjecDocker foloowing this order:

ProjectDocker:

  - Docker.yml 
  
  - Server
      - server.py
      - Server.html
      - DockerFile
      
  - Client
      - client.py
      - DockerFile
      
  - Firewall
      - clean.sh
      - fw.sh
      - DockerFile

# Test the infrastructure

First you will have to use a command to build the docker-compose and compile eveyrthing which is:
      docker-compose build
      
And then you will run you docker with:
      docker-compose up
      
Now you are done and everything is working. You can now test if the firewall is effective and if you can access to the server.
