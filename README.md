# RHEL7-to-RHEL-8

RHEL 7 to RHEL 8 upgrade procedure High-level overview:

The technology stack used:
1. AWS cloud services
2. Linux fundamentals
3. Bash scripting
4. IAC (CloudFormation or Terraform)
5. Knowledge of RHEL 

2 ways to do it:

a. In-place upgrade
b. Fresh Installation - preferred way

a. In-place upgrade:
Blockers you might face: 
subscription-manager: servers are not registered to the RedHat satellite server so you create a developer account and basically register your servers to the RedHat satellite server and then we use the Leapp utility to upgrade- the POC process (This includes the cost associated with the license) - This comes into picture if your servers are not registered.

Steps followed for In-place upgrade:
To start the Upgrade you first need to Disable FIPS(Federal Information Processing Standard) mode 
1. Make sure your servers are connected to the RedHat satellite server - follow your organization's run books to connect to the satellite server
2. Then have your leap utility installed and we will be using this utility to upgrade to RHEL 8
3. So, the next thing you need to do is pre-upgrade check which will give you a report in the text(.txt) format and see if any inhibitors need to be addressed which will prevent from upgrade. To name some of them are:
   a. loaded kernels that are removed in RHEL 8 that are still present in RHEL 7.  So finding out where they are and doing just the required changes would be a bit tricky. 
   b. missing answers in answer files  - you basically need to update some files with necessary answers, so that they won't be a blocker in the process of upgrade
4. Lastly, then upgrade and reboot
Once rebooted then you need to enable FIPS mode back

The important thing before completing this upgrade:  ***Regenerate cryptographic keys

Note: so, the problem with in-place upgrades, there are a lot of dependencies that you need to take care of also the team of architects and security professionals have suggested that there would be a lot of vulnerabilities that we need to address during the upgrade and after and things might be unpredictable. So that was the reason we had to choose a clean installation. 

b. Fresh Installation:

Procedure followed:

1. Considering your Infrastructure is automated and built using Cloud Formation Templates or Terraform (IAC) on AWS Cloud. 
2. Terminologies to keep in mind: Golden AMI and Silver AMI
      Golden AMI - Brand new RHEL 8 image provided by RedHat that is approved as per the organization standards.
      Silver AMI - So based on the application you have to configure and create bash scripts as needed to run web server dependencies i.e (Tomcat, java, python runtimes) and proxy servers i.e ex: Nginx configurations and if you have any content management have php with appropriate versions/releases that are compatible. 
3. Do these steps and once you are ready with your AMI's(Amazon Machine Image) we just need to replace the new AMI id in the CloudFormation or Terraform whichever your organization is using and we need to bring up the new environment. 
4. IMP: ** One thing to make sure of is if for suppose your Infrastructure as code is not up to date with the latest configuration and if you have done any manual changes either to the load balancer or security groups or even in servers. The key point is just need to keep up to date with the existing environment as a reference and we have to try to replicate the same in the new instances. 
5. So, once the whole environment is in place, then navigate to your /app volumes(usually EBS Volumes)(NFS volumes, or wherever your application data resides) on the existing environment from where you basically take the latest snapshot of it and create a new EBS volume with proper mount points and attach it to the servers by properly matching with the existing environment.
6. Once you have your latest /app volumes you just basically log in to the servers and mount it to the system by changing entries in the fstab file. 
7. so, once mounting is done next step is changing the permissions to files and folders, and then bringing up the Tomcat i.e. your application is hosted on a web server.
8. Bringing up Tomcat will require you to write the tomcat.service unit file and make sure the important thing here is SElinux mode(By default enforcing mode) which is the new feature in RHEL 8 that might block Tomcat from running disable it and should be good.
9. Also, make sure that any ssl/tls certificates are used and they are properly added to it correct path. The next steps will all depend on what kind of application your organization what are its prerequisites and steps might change as per the requirement. 
10. so, once you have this in place server-side configurations are done, now we have to check the load balancer side configurations whether the target groups are properly assigned and listener rules are written properly whether the instances attached to target groups are healthy or not, need to check all of these one by one by matching it with the existing environment.
11. After that make sure what port numbers are used, and open it inside the new servers using firewalld. After that match the security group rules and NACL rules if needed as per previous environments.
12. Once you have your application running make sure your database is connected to the application and gather all the required passwords that are needed during the infrastructure provisioning.
13. Once it is up and running handover the server details to the application team for Smoke testing and coordinate with them as and when needed. Once everything is in place map it to the proper DNS record as per your organization's naming convention and now the new environment is up and running.
14. Now slowly we shift our user traffic to the new environment by following proper deployment strategies and move to the new environment as decommission the old environment by following step by step.  
