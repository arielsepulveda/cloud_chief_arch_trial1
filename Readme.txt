 ▄████████    ▄█     █▄    ▄████████
 ███    ███  ███     ███  ███    ███
 ███    ███  ███     ███  ███    █▀
 ███    ███  ███     ███  ███
▀███████████ ███     ███ ▀███████████
 ███    ███  ███     ███          ███
 ███    ███  ███ ▄█▄ ███    ▄█    ███
 ███    █▀    ▀███▀███▀   ▄████████▀

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
CROSS|OVER PROJECT EVALUATION (3 days)
= = = = = = = = = = = = = = = = = = = = = = = = = = = =
by Ariel Sepulveda (cascompany@gmail.com)  -  14/1/2017


INTRODUCTION
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

This is a full high availability deployment with a MyBB example implementation,
that will work without a single point of failure for a maximum uptime.

1.- What is needed to deploy this architecture ?
 *  The provided solution.json file.
 *  An Amazon Web Services Account (with a Key Pair created)

2.- Steps to deploy the solution.json file.
 *  Login to https://aws.amazon.com, or create a new account from here :
    - https://portal.aws.amazon.com/gp/aws/developer/registration/index.html
    following screen instructions.

   - Important: Due to the technologies used on this infrastructure, you must
   - deploy on zones with EFS, or better with EFS and Multi-AZ, regions that
   - have both technologies are :
                                  *  Virginia (US)
                                  *  Ireland (EU)
                                  *  Oregon (US)
   - Click on the top right of the screen, and select one of this zones before
   - continue, Thank you.

 *  Now, From the Console Home, click on Services and then on "CloudFormation"
    (under Management Tools) and Look for the [Create Stack] Blue Button.
 *  When it asks for a template, choose "Upload a template to Amazon S3", click
    [choose file], and point to the solution.json file, and click [Next].
 *  Choose a Stack name (A-Z/0-9, no spaces or special characters)
 *  Select your KeyName, if you dont have one, you need to create a Key Pair,
    follow instructions at :
    - http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html
 *  Enter Administration Email and SSH mask, or leave as 0.0.0.0/0 to allow SSH
    from anywhere.
 *  Select your EC2 instance types, storage size and autoscaling options
 *  Select your Database instance type and storage, you can enable Multi-AZ for
    maximum availability, just make sure you are in the correct AWS Region.
 *  Select the desired Database Name, the Database Username, and the Database
    Password (Note, due to the fact this is a demo, the database password, can
    be used to SSH as root to the instances, on standard port 22)
 *  Select the availabily zones you want to use to deploy your servers (ideally
    select 3 different zones, exept on regions with 2 zones.)
 *  Select the Volume Name, and the Mount Point (please note, this will be also
    used as the Document Root for Apache)
 *  Click [Next], you can specify Tags or Permissions, but they are not needed,
    you can click [Next] again.
 *  Review the build information, scroll, click on the "AWS::IAM" warning
    confirmation box, and then click [Create]
 *  The Process will Start inmediatly, be patient, the build takes around 10 to
    15 minutes to complete.

When it finishes, you will see a "CREATE_COMPLETE" green status, it means that
stack is ready to use.
On "Events" and "Resources" tabs, you can see all build information while it's
being generated, and in "Outputs", when the process is done, you will have more
information, including the "ElasticBalancerURL", you can click on this URL to
access the website created, as well as other information, also look at the
"Parameters" tab, you will found the Username, Password and other parameters
there, please note that the Password, can be used to login by SSH as root, and
also, this same Username and Password, can be used to access MyBB Admin console.

Now, you can inspect on AWS all the services and instances created, please take
in mind, that you will need to "Delete Stack" (From Actions Menu, by selecting
the Stack created) when you finish, because this could generate costs.

AWS Services Deployed, includes :
- VPC (created a custom VPC, networks, subnets, etc)
- EC2 Instances (automatically generated, based on Auto Scaling Group)
- Elastic File System (EFS) to store the Apache Home, mounted as NFS and shared
  across all servers.
- RDS Instance (with Multi-AZ if selected)
- Elastic Load Balancer, with Route53 failover.
- Miscs (Route Tables, Subnets, Security Groups, DHCP, IAM, etc.)


TO DO
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
There are lot of things that can be done, to go even better and better with
this setup, this are some examples of what can be done :

 1.- Add a S3 bucket & CloudFront :
     Copy all static files to the bucket, enable CloudFront for this bucket,
     then rewrite urls of the website to use this files, this will cache all
     this files, in a location near the customer.
 2.- Add AWS ElastiCache :
     This will cache dynamic pages, drastically reducing the load on the MySQL.
 3.- Add RDS Read Replicas :
     This can help by dividing the read load to the SQL engine over more
     servers, also, on different regions, and act as a backup DB in case of
     failure on zones where Multi-AZ is not available.
 4.- Add a S3 bucket to copy & sync the files from the EFS to the bucket, and
     use it as well to sync the files to servers on local EBS file systems.
     This will allow them to run on zones where EFS is not yet available.
 5.- Setup a Bastion Instance, to be the only DMZ server, secure the Web Servers
     to only local traffic, and use exclusively the Elastic Load Balancer to
     access the website.  And the Bastion machine as the only point to SSH to
     the Web Servers.  This would be a very secure configuration, as the Bastion
     would not be a public machine, just for internal use, you can even, setup
     a VPN from the Business Network, so it only can be accessed from there.
 6.- Setup multiple configurations on multiple Regions (each with a setup like
     the one used here) and use Route 53 to send traffic based on latency for
     example, to each Region.


FINAL THOUGHTS
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
I had also setup so the cloudformation creates its DNS in Route 53, if the
domain is previously setup, it will make a "forum.domain.com" zone automatically
that points to the Load Balancer long URL.
I have also tried to minimize the parameters asked, and put as much defaults as
posibly maintaining the "customizability" of the stack.
I have done my best here, I hope you like it, I can get really better if I need
to do this everyday.
Issues I had, well... trying to find how to make management system without
taking me another 3 days, seemed to be impossible for me (Not a DEV really)
Also, the NameServer update times, did not really helped on this part, it gone
pretty straightforward, but it takes time...
At some point, the RDS and other things not worked, mostly because the first
hardcoded stuff (names, urls, login, pass, etc).
Also, a tool for "Creating" without actually creating the things on AWS will
help a lot reduce the times to test/fix/test/fix. Of course, next one, will be
a lot faster developed for sure.


Kind Regards,
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
   _       _     _   ___                _            _
  /_\  _ _(_)___| | / __| ___ _ __ _  _| |_ _____ __| |__ _
 / _ \| '_| / -_) | \__ \/ -_) '_ \ || | \ V / -_) _` / _` |
/_/ \_\_| |_\___|_| |___/\___| .__/\_,_|_|\_/\___\__,_\__,_|
      cascompany@gmail.com   |_|       +34 663429244
