# ELK Stack Part 1

Today I’m going to install and configure the Elastic stack on a virtual network for future use of introducing attack telemetry for analysis and overall practice to configure and fine tune detection rules to decrease noise and false positives.  I will be starting with 2 Linux servers and a Windows host and will add to the network in the future, with plans of incorporating ticketing systems or a tracking tool such as Jira. It is always best practice to take notes while building out and configuring systems. 
<br>
<br>
We’ll begin by installing and configuring Elasticsearch. We want to SSH into our elastic server so we can import the Elasticsearch public GPG key followed by adding Elastic to our “sources.list.d” so APT can search for it when we install and update. 
<br>
<br>
<a href="https://imgur.com/vfqU2yG"><img src="https://i.imgur.com/vfqU2yG.jpg" title="source: imgur.com" /></a>
<br>
<br>
Next, we will run “apt update” and install Elasticsearch. There will be some important information that you want to copy after the installation, this will be the superuser password and other important directories.  
<br>
<br>
<a href="https://imgur.com/0NnoasL"><img src="https://i.imgur.com/0NnoasL.jpg" title="source: imgur.com" /></a>
<br>
<br>
Next, we will start Elasticsearch and then enable it so it starts up automatically when the server is booted. 
<br>
<br>
<a href="https://imgur.com/KxqSjE8"><img src="https://i.imgur.com/KxqSjE8.jpg" title="source: imgur.com" /></a>
<br>
<br>
Now we will run a quick command to see if the elastic service is running correctly. There is currently no SSL certificate, so you have to run –k  with curl to ignore certificates and you must add the password/token we copied earlier during installation to the URL. The JSON data confirms the service is running correctly. 
<br>
<br>
Ie. Curl –X GET –k https://elastic:<token>@localhost:9200 
<br>
<br>
<a href="https://imgur.com/rsSx5Z5"><img src="https://i.imgur.com/rsSx5Z5.jpg" title="source: imgur.com" /></a>
<br>
<br>
Next, we will install Kibana, which will give us the web interface so we can view and analyze the data that we will be collecting.  
<br>
<br>
<a href="https://imgur.com/kTlsVdZ"><img src="https://i.imgur.com/kTlsVdZ.jpg" title="source: imgur.com" /></a>
<br>
<br>
Before we can start and enable Kibana, we need to generate the enrollment token for the system. This information was shared during the Elasticsearch install, which you should’ve taken note of. After the token is generated, we will add it to the kibana-setup binary, followed by starting and enabling Kibana.  
<br>
<br>
<a href="https://imgur.com/XMQnnnh"><img src="https://i.imgur.com/XMQnnnh.jpg" title="source: imgur.com" /></a>
<br>
<br>
We will now install NGINX on the same server and since this is a server on a private VM, I’m not going to be installing an SSL certificate, we will use a proxy pass instead. (Credit: IppSec) 
<br>
<br>
After that information is saved, we will restart and enable nginx. 
<br>
<br>
<a href="https://imgur.com/0gXFtbg"><img src="https://i.imgur.com/0gXFtbg.jpg" title="source: imgur.com" /></a>
<br>
<br>
<a href="https://imgur.com/zpTOvhr"><img src="https://i.imgur.com/zpTOvhr.jpg" title="source: imgur.com" /></a>
<br>
<br>
If we open a browser and put in the IP address of the server Elastic was installed on it will open a portal. The username will be “elastic” and the password is the one we recorded during the beginning of the installation. After that we will be adding integrations.  
<br>
<br>
<a href="https://imgur.com/BYN9Cql"><img src="https://i.imgur.com/BYN9Cql.jpg" title="source: imgur.com" /></a>
<br>
<br>
We will search for Fleet Server and at that first and I will keep all the default settings for now and click “save and continue”.  After that it wants us to add an Elastic agent to our fleet server, so I will SSH into the server and configure it. In Kibana I will go through the steps to add a Fleet Server Host and generate a service token. After that is done it will give you a command to execute on the server. 
<br>
<br>
<a href="https://imgur.com/kUXaMTp"><img src="https://i.imgur.com/kUXaMTp.jpg" title="source: imgur.com" /></a>
<br>
<br>
<a href="https://imgur.com/3O6ZT4v"><img src="https://i.imgur.com/3O6ZT4v.jpg" title="source: imgur.com" /></a>
<br>
<br>
Before I run the command on the Fleet server, I will copy /etc/elasticsearch/certs/http_ca.crt to a new directory /usr/local/etc/ssl/certs/elastic/. After copying and pasting the command to execute in Kibana, at the end of the command we have to add in our SSL certificate. So it should look like: “ / --fleet-server-es-ca=/usr/local/etc/ssl/certs/elastic/http_ca.crt --insecure “ 
<br>
<br>
<a href="https://imgur.com/KlJQTTk"><img src="https://i.imgur.com/KlJQTTk.jpg" title="source: imgur.com" /></a>
<br>
<br>
Now we will create a second policy and add a fleet to our Windows box. We will open PowerShell with administrator rights and copy the script from Kibana and execute it.  It is like the steps from the first fleet installation. So, what the script is doing is its downloading the installation data, unzipping it to a directory, switching to that directory, and launching the installation executable. The only thing that I had to add was “--insecure”, because we’re using self-signed certificates. After installation, we get confirmation over on Kibana.  
<br>
<br>
<a href="https://imgur.com/eMGqnuY"><img src="https://i.imgur.com/eMGqnuY.jpg" title="source: imgur.com" /></a>
<br>
<br>
I decided to reboot my machines and upon restart I noticed that both of my fleet agents were offline. After a decent amount of research and troubleshooting, I found that the agents were outputting data to a different IP address and also my fleet host URL needed to be changed. So, I had to go into the kibana.yml and change two settings both “is defaut” and  “is_default_monitoring” to false.and then go to the settings tab in the Kibana Fleet GUI to change the IP addresses there. After doing that, for some reason the default output kept populating, so I added a new output and set it to default for both agent integration and agent monitoring. I also had to add the line “ssl.verification_mode: “none”” due to us using a self-signed certificate. 
<br>
<br>
<a href="https://imgur.com/15Wkc9o"><img src="https://i.imgur.com/15Wkc9o.jpg" title="source: imgur.com" /></a>
<br>
<br>
<a href="https://imgur.com/dD97b6A"><img src="https://i.imgur.com/dD97b6A.jpg" title="source: imgur.com" /></a>
<br>
<br>
Next, we will add another integration and this time it will be Elastic Defend which focuses on prevention, detection and response capabilities across traditional endpoints and cloud environments. It gives you the options of data collection, next-gen antivirus, essential or complete EDR. When they asked where to add this integration, I added it to an existing policy due to the size of my network and resources constraints.  
<br>
<br>
<a href="https://imgur.com/ub1YGNj"><img src="https://i.imgur.com/ub1YGNj.jpg" title="source: imgur.com" /></a>
<br>
<br>
We’re receiving data, it’s just not that exciting due to it only being me accessing the boxes. I’m going to do part two we’re I will tune the system and I also plan on adding a web application to the server which I will attack and later analyze the data and see how the EDR responds.  
<br>
<br>
<a href="https://imgur.com/3vXGBuV"><img src="https://i.imgur.com/3vXGBuV.jpg" title="source: imgur.com" /></a>
<br>
<br>
Lessons learned? Read the manuals! So many headaches can be avoided. Also taking meticulous notes while doing installs or changes to systems. Documentation is very important during the change management process and keeping neat and tidy notes will increase productivity and save time.  




























































