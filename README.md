# Installing-Jenkins-with-SSL
Installing Jenkins with SSL  (https://imdurgadas.medium.com/installing-jenkins-with-ssl-96d8cf256b6f)

Jenkins is an opensource automation server written in Java which helps to automate the non-human part of the Software Development process with Continuous Integration and facilitating technical aspects of Continuous Delivery.

Jenkins Installation
There are different ways by which we can install Jenkins:
Jenkins is available in 2 release modes:

1. Long Term Support (LTS) — This is the stable version and is recommended.
2. Weekly release — This is released weekly and includes bug fixes, features etc.
We will be using the LTS version in this article.
Pre-requisites : You need to have Java installed and should be available in the PATH.
Installation using Docker
This is the preferred way as it doesn’t matter which Operation System you use. Execute the docker run command and that would set it up for you :-)
docker run --name jenkins -p 8080:8080 -p 50000:50000 -d jenkins/jenkins:lts
docker tail -f jenkins
2. Installation using Repository [ Ubuntu and Red Hat Enterprise Linux ]
For Ubuntu:
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
echo "deb http://pkg.jenkins.io/debian-stable binary/" >> /etc/apt/sources.list
apt-get update && apt-get install -y jenkins
For Red Hat Enterprise Linux:
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
yum install jenkins
3. Using WAR
wget -L http://mirrors.jenkins.io/war-stable/latest/jenkins.war
java -jar jenkins.war
Once installed using any of the above approaches, Jenkins should be up and running on port 8080.
http://localhost:8080


Jenkins Configuration
Jenkins configuration file is located at : /etc/default/jenkins
Increase the heap size : Edit the /etc/default/jenkins and the below configuration-
JAVA_ARGS="-Xmx2048m" #Where 2048m is 2GB of heap. Default is 256MB.
Configure https:
# Create key and certificate. Provide information wherever asked.

openssl req -newkey rsa:2048 -nodes -keyout jenkins.key -x509 -days 700 -out jenkins.crt

#Below command would ask you for password. Remember the same as that will be used during the configuration.

openssl pkcs12 -inkey jenkins.key -in jenkins.crt -export -out keys.pkcs12

keytool -importkeystore -srckeystore keys.pkcs12 -srcstoretype pkcs12 -destkeystore /var/lib/jenkins/jenkins.jks

Once the certificates are generated , configure the same in the /etc/default/jenkins file.

JENKINS_ARGS="--webroot=/var/cache/$NAME/war --httpPort=-1 --httpsPort=8443 --httpsKeyStore=/var/lib/jenkins/jenkins.jks --httpsKeyStorePassword=<password>"

Note: httpPort=-1 would disable http. If you wish to have both http and https , then provide the desired port.
Once the above configuration is done, restart jenkins to reflect the change.
Now, you will be able to access jenkins via https using the below link

 https://localhost:8443


**To configure the nexus to work with SSL , we will use the same certificate which we created for jenkins:**

We are usig nexus-3 use below link to download and install it:

https://devopscube.com/how-to-install-latest-sonatype-nexus-3-on-linux/

Add below part to **"jetty-https.xm"**

[root@ip-172-31-71-214 jetty]# pwd
/opt/nexus/etc/jetty
[root@ip-172-31-71-214 jetty]# ll jetty-https.xml
-rw-r--r-- 1 ec2-user ec2-user 3619 Apr 24 14:24 jetty-https.xml

    <Set name="KeyStorePath"><Property name="ssl.etc"/>/jenkins.jks</Set>
    <Set name="KeyStorePassword">tushar123</Set>
    <Set name="KeyManagerPassword">tushar123</Set>
    <Set name="TrustStorePath"><Property name="ssl.etc"/>/jenkins.jks</Set>
    <Set name="TrustStorePassword">tushar123</Set>
    <Set name="EndpointIdentificationAlgorithm"></Set>

Need to copy keystore from "/var/lib/jenkins/jenkins.jks" to "/opt/nexus/etc/ssl/jenkins.jks"

Now do below changes to /opt/sonatype-work/nexus3/etc/nexus.properties

# cat /opt/sonatype-work/nexus3/etc/nexus.properties
# Jetty section
application-port=8081
application-port-ssl=8444
application-host=0.0.0.0
nexus-args=${jetty.etc}/jetty.xml,${jetty.etc}/jetty-http.xml,${jetty.etc}/jetty-https.xml,${jetty.etc}/jetty-requestlog.xml
nexus-context-path=/

# Nexus section
nexus-edition=nexus-pro-edition
nexus-features=\
nexus-pro-feature

# nexus.hazelcast.discovery.isEnabled=true



