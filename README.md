<h1>Offense-Defense Lab: Reverse Shell Exploitation & Detection</h1>


<h2>Description</h2>

This project builds a small, production-style environment that enforces least privilege, traceable access, and a reviewable audit trail for two teams (Project-A and Project-B). You’ll provision users and groups, lock down shared storage with Linux permissions and ACLs, and harden collaboration behavior with the sticky bit and default ACLs. To support accountability, you’ll tailor shell history policies (shorter for senior analysts, longer for others) and enable auditdto watch the /home/project workspace for reads/writes/execs/attribute changes. Finally, you’ll stand up an Apache web dashboard protected by Basic Auth that periodically exports and publishes audit findings (via cron) so IT can monitor and investigate access attempts from a browser.

<br />

<h2>Languages and Utilities Used</h2>

The project primarily uses specialized penetration testing and analysis software, leveraging operating system utilities and command-line tools.

- Bash / Shell scripting — user/group management, ACLs, services, cron, and log export.
- POSIX permissions & ACLs — chmod, sticky bit, and fine-grained setfacl/getfacl.
- HTTP/HTTPS — serving the audit dashboard via Apache; HTTP Basic Auth with .htaccess/.htpasswd.
- Core Linux admin tools: groupadd, useradd, passwd, chown, chmod, chsh.
= ACL tools: setfacl, getfacl (grant per-user access and set default ACLs).
- Audit stack: auditd, auditctl, ausearch (watch /home/project and query events).
- Service control: systemctl (start/enable auditd & apache2).
- Web server: Apache2 (serve auditlog.txt with access control).
- Auth utility: htpasswd (create credential file for Basic Auth).
- Automation: cron (periodic ausearch export to the web directory).
- Editors & CLI: nano, standard shell utilities.

<h2>Environments Used </h2>

-	OS: Debian/Ubuntu-based Linux host (compatible with Kali) for all administrative and server tasks.
-	Filesystem layout: project data at /home/project; web content under /var/www/html.
-	Web access point: http://localhost/auditlog.txt protected by Basic Auth for IT review.
-	Users & groups for testing: pa1…pa5 (Project-A) and pb1…pb3 (Project-B) with tailored history sizes and ACLs to validate segregation and auditing.

<h2>Step-by-Step Guide:</h2>

Step 1 - Create user accounts for Project-A and Project-B groups, configuring ACLs to restrict file access to owners only.

1.	Create Project-A group, open the Terminal and run:

sudo groupadd proja

<img src="https://i.imgur.com/Y3MgJiI.png"/>
<br />
<br />
2.	Create Project-B group, run:

sudo groupadd projb

<br/>
<img src="https://i.imgur.com/5cizFJ0.png"/>
<br />
<br />
3.	Create users and assign them to Project-A group. In this case the name of the users will be PA1, PA2, PA3, PA4 and PA5, run:

sudo useradd -m -g proja pa1 

sudo useradd -m -g proja pa2 

sudo useradd -m -g proja pa3 

sudo useradd -m -g proja pa4 

sudo useradd -m -g proja pa5

<br/>
<img src="https://i.imgur.com/UtYvkLO.png"/>
<br />
<br />
4.	Create users and assign them to Project-B group. In this case the name of the users will be PB1, PB2 and PB3, run:

sudo useradd -m -g projb pb1 

sudo useradd -m -g projb pb2 

sudo useradd -m -g projb pb3

<br/>
<img src="https://i.imgur.com/TdEOLlJ.png"/>
<br />
<br />
5.	Set passwords for each user on Project-A group, run:

sudo passwd pa1 

sudo passwd pa2

sudo passwd pa3 

sudo passwd pa4 

sudo passwd pa5

<br/>
<img src="https://i.imgur.com/oKUCMfd.png"/>
<br />
<br />
6.	Set passwords for each user on Project-B group, run:

sudo passwd pb1 

sudo passwd pb2 

sudo passwd pb3

<br/>
<img src="https://i.imgur.com/16jiNo9.png"/>
<br />
<br />
<br />
7.	Set a centralized storage. Create a location folder for both projects, run:

sudo mkdir /home/project

<br/>
<img src="https://i.imgur.com/WECnDsJ.png"/>
<br />
<br />
8.	Change directory to the folder you’ve just created:

cd /home/project

<br/>
<img src="https://i.imgur.com/6J67wFV.png"/>
<br />
<br />
9.	Assign Project-A group ownership to the directory, run:

sudo chown :proja /home/project

<br/>
<img src="https://i.imgur.com/eJ69EH5.png"/> 
<br />
<br />
10.	Assign Project-B group ownership to the same directory, run:

sudo chown :projb /home/project

<br/>
<img src="https://i.imgur.com/GpZQsew.png"/>
<br />
<br />
11.	Set permissions on the project directory. Run the following command to give full access (read, write, and execute) to the owner and group, but no access to others:

sudo chmod 770 /home/project

<br/>
<img src="https://i.imgur.com/BgspPGH.png"/>
<br />
<br />
12.	Grant specific user permissions with ACL (Access Control List). Run the following command to give users pa1, pa2, pa3, pa4 and pa5 on Project-A group full permissions (read, write, and execute) on the /home/project directory:

sudo setfacl -m u:pa1:rwx /home/project

sudo setfacl -m u:pa2:rwx /home/project 

sudo setfacl -m u:pa3:rwx /home/project 

sudo setfacl -m u:pa4:rwx /home/project 

sudo setfacl -m u:pa5:rwx /home/project

<br/>
<img src="https://i.imgur.com/OYFJCv2.png"/>
<br />
<br />
13.	Repeat the same steps for users pb1, pb2 and pb3 on Project-B group, run:

sudo setfacl -m u:pb1:rwx /home/project 

sudo setfacl -m u:pb2:rwx /home/project 

sudo setfacl -m u:pb3:rwx /home/project

<br/>
<img src="https://i.imgur.com/cG6lg3m.png"/>
<br />
<br />
14.	Modify group permissions with ACL. Run the following command to give the group read
(r) and execute (x) permissions on the /home/project directory, while removing write (w) permission:

sudo setfacl -m g::r-x /home/project

<br/>
<img src="https://i.imgur.com/oif3XIs.png"/>
<br />
<br />
15.	Set default ACL for user permissions. Run the following command to ensure that any new files or directories created inside /home/project will automatically give the owner (user) full permissions (rwx):

sudo setfacl -d -m u::rwx /home/project

<br/>
<img src="https://i.imgur.com/LQZucxC.png"/>
<br />
<br />
16.	Restrict default ACL permissions for others. Run the following command to ensure that other users (not part of the owner or group) have no permissions by
defaulton /home/project:

sudo setfacl -d -m o::--- /home/project

<br/>
<img src="https://i.imgur.com/b7zUiEO.png"/>
<br />
<br />
17.	Enable the Sticky Bit on the directory. Run the following command to apply the sticky bit on /home/project:

sudo chmod +t /home/project

<br/>
<img src="https://i.imgur.com/pyP1Fdz.png"/>
<br />
<br />
18.	Change the default shell of the users on Project-A and Project-B groups. Run the following commands to change the login shell of the users to /bin/bash:

sudo chsh -s /bin/bash pa1	  

sudo chsh -s /bin/bash pa2

sudo chsh -s /bin/bash pa3	 

sudo chsh -s /bin/bash pa4

sudo chsh -s /bin/bash pa5	  

sudo chsh -s /bin/bash pb1

sudo chsh -s /bin/bash pb2	

sudo chsh -s /bin/bash pb3

<br />
<img src="https://i.imgur.com/MyH5gnI.png"/>
<br />
<br />
Step 2 - Set senior analysts to view their last 10 commands and other users to retain last 50 commands for audits.

1.	Considering users pa1 and pa5 user as senior analysts, run the following commands to configure the shell history size to 10 commands:

echo "HISTSIZE=10" | sudo tee -a /home/pa1/.bashrc 

echo "HISTSIZE=10" | sudo tee -a /home/pa5/.bashrc

<br />
<img src="https://i.imgur.com/JvImUVp.png"/>
<br />
<br />
2.	Run the following commands to set the history size to 50 commands for all the other users:

echo "HISTSIZE=50" | sudo tee -a /home/pa2/.bashrc 

echo "HISTSIZE=50" | sudo tee -a /home/pa3/.bashrc 

echo "HISTSIZE=50" | sudo tee -a /home/pa4/.bashrc 

echo "HISTSIZE=50" | sudo tee -a /home/pb1/.bashrc 

echo "HISTSIZE=50" | sudo tee -a /home/pb2/.bashrc 

echo "HISTSIZE=50" | sudo tee -a /home/pb3/.bashrc

<br />
<img src="https://i.imgur.com/TOTW7gj.png"/>
<br />
<br />
Step 3 - Use Syslog/Rsyslog to log unauthorized access attempts, ensuring secure storage for IT review.

1.	Update the package list. Run the following command to refresh the package index on the system:

sudo apt update
 
<br/>
<img src="https://i.imgur.com/KqSMY80.png"/>
<br />
<br />
2.	Install the Auditd service. Run the following command to install auditd, which is the
Linux Audit Daemon used to track security-related events:

sudo apt-get install auditd -y

<br/>
<img src="https://i.imgur.com/1bzuDAV.png"/>
<br />
<br />
3.	Add an audit rule to monitor the /home/project directory. Run the following command to configure auditd to watch all activity in the /home/project directory:

sudo auditctl -w /home/project -p rwxa -k project_access

<br/>
<img src="https://i.imgur.com/R3j5liw.png"/>
<br />
<br />
4.	Start the auditd service, run:

sudo systemctl start auditd

<br/>
<img src="https://i.imgur.com/x1ULSV0.png"/>
<br />
<br />
5.	Enable the auditd service at startup. Run the following command to ensure that the audit daemon (auditd) starts automatically whenever the system boots:

sudo systemctl enable auditd

<br/>
<img src="https://i.imgur.com/HBCUbjM.png"/>
<br />
<br />
6.	Verify the auditd service status. After enabling auditd, check if the service is currently running with:

sudo systemctl status auditd

<br/>
<img src="https://i.imgur.com/VClTQKp.png"/>
<br />
<br />
<br />
7.	Edit audit rules configuration. To add custom audit rules that define what events should be logged, open the audit rules file with:

sudo nano /etc/audit/rules.d/audit.rules

<br/>
<img src="https://i.imgur.com/Jy3WybP.png"/>
<br />
<br />
8.	Add an audit rule for /home/project directory. To monitor all access attempts (read, write, execute, and attribute changes) to the /home/project directory, add the following line inside the audit rules file:

-w /home/project -p rwxa -k project_access

<br/>
<img src="https://i.imgur.com/AhWLU1l.png"/>
<br />
<br />
9.	Restart the service to make sure the changes are working, run:

sudo systemctl restart auditd

<br/>
<img src="https://i.imgur.com/zuxKHn6.png"/>
<br />
<br />
10.	Verify audit logs for /home/project access. Now that the audit rule has been set, you can search the logs to confirm that all activity in the /home/project directory is being tracked. Run:

sudo ausearch -k project_access

<br/>
<img src="https://i.imgur.com/UQFufWj.png"/>
<br />
<br />
Step 4 - Develop a web dashboard for IT teams to monitor and analyze security violations.

1.	Set up the web server using apache2, run: sudo apt-get install apache2

<br/>
<img src="https://i.imgur.com/kIwrGKC.png"/>
<br />
<br />
2.	Export audit logs to a file in the web server directory. Switch to the root user and run the following command to save the audit logs into a text file that can be accessed via the web server:

sudo su

<img src="https://i.imgur.com/z9ylCUv.png"/>

ausearch -k project_access >> /var/www/html/auditlog.txt

<img src="https://i.imgur.com/bi2Zshs.png"/>

<br />
<br />
3.	Automate audit log export with a cron job. To ensure that audit logs are regularly exported to the web server directory, edit the root user’s crontab file, run:

crontab -e

Choose 1:

<img src="https://i.imgur.com/UjN0Lbg.png"/>

<br />
<br />
4.	Schedule automated audit log exports. To ensure the audit logs are exported regularly, add a cron job that runs every 5 minutes. In the nano screen that opens add the following rule at the bottom:

*/5 * * * * ausearch -k project_access >> /var/www/auditlog.txt

<img src="https://i.imgur.com/YURqBLo.png"/>

<br />
<br />
5.	Configure Apache to serve the audit log. Open the Apache default site configuration file:

sudo nano /etc/apache2/sites-available/000-default.conf

<img src="https://i.imgur.com/OqSP9Yb.png"/>

<br />
<br />
6.	Enable Web Access to the Logs Directory. To make the audit log (auditlog.txt) accessible through a web browser, you need to allow Apache to read .htaccessfiles
in /var/www/html. At the bottom of the file, add the following block:


<Directory /var/www/html> 

AllowOverride All
</Directory>

<br />
<br />
<img src="https://i.imgur.com/F9eSGVL.png"/>
<br />
<br />
7.	Protect Web Access with Authentication. To make sure only authorized users can view the audit logs from a browser, configure Basic Authentication using an .htaccess file. Create or edit the .htaccess file inside the web root:

sudo nano /var/www/html/.htaccess

<img src="https://i.imgur.com/MgFacdv.png"/>

Add the following configuration:

AuthType Basic

AuthName "Restricted Access" 

AuthUserFile /var/www/html/.htpasswd 

Require valid-user

<img src="https://i.imgur.com/ZiGI4lt.png"/>

<br />
<br />
8.	Create a Password File for Web Authentication. To secure access to the audit log through the browser, you must create a password file (.htpasswd) that stores the credentials. Run the following command:

sudo htpasswd -c /var/www/html/.htpasswd admin

<img src="https://i.imgur.com/87oeD0d.png"/>

Note: This code will generate the username as admin.
<br />
<br />
9.	Start the Apache2 Web Server. After configuring authentication and preparing the log file, start the Apache2 service so the web server is active:

sudo systemctl start apache2

<img src="https://i.imgur.com/RrmiSLF.png"/>

<br />
<br />
10.	Enable Apache2 to Start on Boot. To ensure the Apache2 web server runs automatically after each reboot, enable it as a persistent service:

sudo systemctl enable apache2

<img src="https://i.imgur.com/t9fX0gq.png"/>

<br />
<br />
11.	Switch to User pa1. To test the permissions and access controls you set earlier, switch to the user pa1 with the following command:

su – pa1

<img src="https://i.imgur.com/kPbj9vK.png"/>

<br />
<br />
12.	Test File Creation as User pa1. Now that you switched to user pa1, try creating a new file inside the /home/project directory:

touch /home/project/testfile.txt

<img src="https://i.imgur.com/0PVCe2h.png"/>

<br />
<br />
13.	Test File Deletion as User pa2. Switch to user pa2 and attempt to delete the file testfile.txt created earlier:

su – pa2

rm /home/project/testfile.txt

<img src="https://i.imgur.com/Xjzeky1.png"/>

<br />
<br />
14.	Exit pa2 user:

exit

<img src="https://i.imgur.com/xmeZgO2.png"/>

<br />
<br />
15.	View Command History for User pa1. Run the following command:

history

<img src="https://i.imgur.com/bTKBzL2.png"/>

<br />
<br />
16.	Exit the pa1 user and search the Audit Logs for Project Access Events. Run the following command:

exit

sudo ausearch -k project_access

<img src="https://i.imgur.com/ueJwSaX.png"/>

<img src="https://i.imgur.com/au8jqCu.png"/>
<br />
<br />
17.	Access the Audit Logs via Web Browser. Open your web browser and in the address bar, type:

http://localhost/auditlog.txt

<img src="https://i.imgur.com/1dQNFwP.png"/>
<br />
<br />
18.	Because we configured .htaccess authentication earlier, the browser will prompt you for a username and password. Enter the credentials you created with
the htpasswd command (in this case, admin as the username and the password you set). After authentication, you will be granted access to the auditlog.txt file directly in the browser:

<img src="https://i.imgur.com/lJV2mKl.png"/>

<img src="https://i.imgur.com/UElYIrw.png"/>
<br />
<br />

Conclusion:

This project delivered a small, production-style solution for secure, segregated file storage with accountable access for two project teams. We enforced least-privilege at the filesystem layer using Linux groups, permissions, the sticky bit, and fine-grained ACLs so that Project-A and Project-B users can only read/write what they own, while preventing cross-project access and accidental deletions in shared areas. We complemented access control with auditd rules that watch the project directory for reads, writes, execution, and attribute changes, producing an evidence trail that ties actions to individual users. For visibility, the workflow exports selected audit events to a read-only log and publishes it behind Apache with Basic Auth, giving managers a simple, authenticated way to review activity without granting shell access.

The end state meets the core objectives: clear separation of duties, traceability of every sensitive action, and a lightweight reporting path that is easy to operate. The approach is repeatable on any Debian/Ubuntu-based system and relies only on standard tools, which keeps maintenance cost low and makes handoff straightforward.




<br /></p>

<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
