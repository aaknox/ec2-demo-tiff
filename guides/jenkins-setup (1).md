# Jenkins EC2 Setup
this will explain how to set up your jenkins ci/cd pipeline on your EC2.
## steps
1. ssh into your ec2 instance
2. sudo yum update
3. sudo yum install git

4. sudo yum install -y java-1.8.0-openjdk-devel 
    - to install Java 11 you need to use the extra repository provided by amazon: `sudo amazon-linux-extras install java-openjdk11`
5. wget https://downloads.apache.org/tomcat/tomcat-8/v8.5.81/bin/apache-tomcat-8.5.81.tar.gz
 (make sure to replace this with the updated version)
6. tar -zxvf apache-tomcat-9.0.41.tar.gz
    - unzip, extract, verbose, 
    
    tar -zxvf apache-tomcat-8.5.81.tar.gz

7. go to conf folder
8. open tomcat-users.xml in your text editor of choice (vi/vim, nano)
9. add role and user (username and password are up to you)
    ```xml
    <role rolename="manager-script" />
    <user username="tomcat" password="tomcat" roles="manager-script" />
    <role rolename="manager-gui" />
    <user username="tom" password="cat" roles="manager-gui" />
    ```
10. go to webapps/manager/META-INF
11. open context.xml in your text editor of choice
12. add the EC2 IP addresses to the Valve element
    - BEFORE:   
        - `<Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />`
    - AFTER: (replace with your EC2's actual IPs, found on EC2 page)
        - `<Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1|EC2PUBLICIP|EC2PRIVATEIP" />`
13. go to tomcat bin folder
14. sudo sh startup.sh
    - this starts up tomcat
    - you can go to aws-hostname:8080 in your browser to see that tomcat is running
    - if it does not - make sure your Security Group has Custom TCP port 8080 rules to allow inbound traffic from anywhere (or your IP address)
15. go to webapps folder
16. wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
wget https://mirrors.jenkins.io/war-stable/latest/jenkins.war
17. in your browser, go to aws-hostname:8080/jenkins
18. follow basic setup prompts, you will need to find a passcode in your ec2
    - sudo su
        - this command will let you view the hidden file
19. install deploy to container plugin
20. in jenkins configuration, set up maven install
21. create a job
22. configure git info, gitscm polling w/ hooks
23. invoke top-level maven targets
    - make sure to set up maven install in jenkins configuration
    - set pom location
    - build steps: clean test package
24. in your github repo, add webhook payload
    - aws-hostname:8080/jenkins/github-webhook/
        - if you forget the '/' at the end it doesn't like that
25. make sure you have an Custom TCP port 8080 security rule for your IP (or for any IPv4 and IPv6) if you didn't already add one, plus add the following (github webhooks):
    - 192.30.252.0/22
	- 185.199.108.0/22
	- 140.82.112.0/20
    - 143.55.64.0/20
    - you can find these IPs on api.github.com/meta under the hooks key
26. push your maven project to your repository
27. check jenkins to make sure the build was successful
28. go to aws-hostname:8080/ProjectName/endpoint on your browser or send a request through postman to see that your application is deployed!