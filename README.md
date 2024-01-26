# Setting-up-IPTABLES-on-Ubuntu-20.x-Server
Iptables is a firewall, installed by default on many Linux distributions.  This walk through is simple for setting up the basic server. In this tutorial we will go over how to set up iptables for the first time.  

Note: When working with firewalls, do not block SSH communication; lock yourself out of your own server (port 22, by default).  

Prerequisites:  

Install iptables persistent to save iptables  

sudo apt install iptables-persistent 
 

make a directory /etc/iptables  

sudo mkdir /etc/iptables  
useful commands:  

sudo iptables -L 
sudo iptables -L -v 
sudo iptables -S  
   

Introducing new rules:  

To begin using iptables, add the rules for authorized inbound traffic for the services you need. Iptables can keep track of the connection’s state. Therefore, use the command below to enable established connections to continue.  

sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
Allow traffic to a specific port to permit SSH connections by doing the following:  

sudo iptables -A INPUT -p tcp --dport ssh -j ACCEPT
Change the input policy to drop once you’ve added all the required authorized rules.  

Note: Changing the default rule to drop will allow only specially allowed connections. Before modifying the default rule, ensure you’ve enabled at least SSH, as stated above.  

sudo iptables -P INPUT DROP 
 

Rules for saving and restoring  

If you restart your server, all these iptables configurations will be lost. Save the rules to a file to avoid this.  

sudo iptables-save > /etc/iptables/rules.v4  
 

You may then just read the stored file to restore the saved rules.  

# Overwrite the existing rules 
sudo iptables-restore < /etc/iptables/rules.v4 
# Append new rules while retaining the existing ones 
sudo iptables-restore -n < /etc/iptables/rules.v4
 

You may automate the restore procedure upon reboot by installing an extra iptables package that handles the loading of stored rules. Use the following command to do this.  

sudo apt-get install iptables-persistent
After installation, the first setup will prompt you to preserve the current IPv4 and IPv6 rules; choose Yes, and press Enter for both.  

   

Saving Updates  

If you ever think of updating your firewall and want the changes to be durable, you must save your iptables rules.  

This command will help save your firewall rules:  

sudo netfilter-persistent save
   

Example of IPTABLES:  

*filter  
:INPUT ACCEPT [63:3208]  
:FORWARD ACCEPT [0:0]  
:OUTPUT ACCEPT [36:2160]  
-A INPUT -i lo -j ACCEPT  
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT (Accepting all established connections)  
-A INPUT -m conntrack --ctstate INVALID -j DROP (Drop statement if service or port doesn't match)  
-A INPUT -s 83.97.73.245/32 -j DROP (Example of blocking bad IP)  
-A INPUT -s 83.97.73.245/32 -j REJECT --reject-with icmp-port-unreachable (Rejecting a bad IP)  
-A INPUT -s 192.168.1.0/24 -p tcp -m tcp --dport 22 -m conntrack --ctstate NEW,> (Allow SSH from subnet)  
-A INPUT -p udp -m udp --dport 137 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT (Allow port)  
-A INPUT -p udp -m udp --dport 138 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT 
-A INPUT -p tcp -m tcp --dport 139 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT  
-A OUTPUT -o lo -j ACCEPT  
 
-A OUTPUT -m conntrack --ctstate ESTABLISHED -j ACCEPT  
 
COMMIT  
 

To accept all traffic on your loopback interface, run these commands:  

sudo iptables -A INPUT -i lo -j ACCEPT 
sudo iptables -A OUTPUT -o lo -j ACCEPT
image.gif 

Allowing Established and Related Incoming Connections  

As network traffic generally needs to be two-way – incoming and outgoing – to work properly, it is typical to create a firewall rule that allows established and related incoming traffic, so that the server will allow return traffic for outgoing connections initiated by the server itself. This command will allow that:  

sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
   

Allowing Established Outgoing Connections  

You may want to allow outgoing traffic of all established connections, which are typically the response to legitimate incoming connections. This command will allow that:  

sudo iptables -A OUTPUT -m conntrack --ctstate ESTABLISHED -j ACCEPT
 

Allowing Internal Network to access External  

Assuming eth0 is your external network, and eth1 is your internal network, this will allow your internal to access the external:  

sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
 

Dropping Invalid Packets  

Some network traffic packets get marked as invalid. Sometimes it can be useful to log this type of packet but often it is fine to drop them. Do so with this command:  

sudo iptables -A INPUT -m conntrack --ctstate INVALID -j DROP
image.gif 

 

Blocking an IP Address  

To block network connections that originate from a specific IP address, 203.0.113.51 for example, run this command:  

sudo iptables -A INPUT -s 203.0.113.51 -j DROP
In this example, -s 203.0.113.51 specifies a source IP address of “203.0.113.51”. The source IP address can be specified in any firewall rule, including an allow rule.  

If you want to reject the connection instead, which will respond to the connection request with a “connection refused” error, replace “DROP” with “REJECT” like this:  

sudo iptables -A INPUT -s 203.0.113.51 -j REJECT
   

Blocking Connections to a Network Interface  

To block connections from a specific IP address, e.g. 203.0.113.51, to a specific network interface, e.g. eth0, use this command:  

iptables -A INPUT -i eth0 -s 203.0.113.51 -j DROP
This is the same as the previous example, with the addition of -i eth0. The network interface can be specified in any firewall rule, and is a great way to limit the rule to a particular network.  

Service: SSH  

If you’re using a server without a local console, you will probably want to allow incoming SSH connections (port 22) so you can connect to and manage your server. This section covers how to configure your firewall with various SSH-related rules.  

Allowing All Incoming SSH  

To allow all incoming SSH connections run these commands:  

sudo iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT 
sudo iptables -A OUTPUT -p tcp --sport 22 -m conntrack --ctstate ESTABLISHED -j ACCEPT
The second command, which allows the outgoing traffic of established SSH connections, is only necessary if the OUTPUT policy is not set to ACCEPT.  

 

Allowing Incoming SSH from Specific IP address or subnet  

To allow incoming SSH connections from a specific IP address or subnet, specify the source. For example, if you want to allow the entire 203.0.113.0/24 subnet, run these commands:  

sudo iptables -A INPUT -p tcp -s 203.0.113.0/24 --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT 
sudo iptables -A OUTPUT -p tcp --sport 22 -m conntrack --ctstate ESTABLISHED -j ACCEPT
The second command, which allows the outgoing traffic of established SSH connections, is only necessary if the OUTPUT policy is not set to ACCEPT.  

Here's a sample of setting up a rule which only allows SSH from a single IP:

Add a new "allow SSH from 1.2.3.4" rule: 

iptables -A INPUT -p tcp -s 1.2.3.4 --dport 22 -j ACCEPT
Block SSH from all other IPs:

iptables -A INPUT -p tcp -s 0.0.0.0/0 --dport 22 -j DROP
Now your INPUT chain will look like:

Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  1.2.3.4              0.0.0.0/0            tcp dpt:22
DROP       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:22
Later, if you need to whitelist a second IP you can use the -I parameter to place it before the blacklist rule.

iptables -I INPUT 2 -p tcp -s 4.3.2.1 --dport 22 -j ACCEPT
 

Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  1.2.3.4              0.0.0.0/0            tcp dpt:22
ACCEPT     tcp  --  4.3.2.1              0.0.0.0/0            tcp dpt:22
DROP       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:22
Notice that using -I INPUT 2 added the new rule as rule number 2 and bumped the DROP rule to number 3.

 

Allowing Outgoing SSH  

If your firewall OUTPUT policy is not set to ACCEPT, and you want to allow outgoing SSH connections—your server initiating an SSH connection to another server—you can run these commands:  

sudo iptables -A OUTPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT 
sudo iptables -A INPUT -p tcp --sport 22 -m conntrack --ctstate ESTABLISHED -j ACCEPT  
 

 

 Allowing Incoming Rsync from Specific IP Address or Subnet  

 

Rsync, which runs on port 873, can be used to transfer files from one computer to another.  

To allow incoming rsync connections from a specific IP address or subnet, specify the source IP address and the destination port. For example, if you want to allow the entire 203.0.113.0/24 subnet to be able to rsync to your server, run these commands:  

sudo iptables -A INPUT -p tcp -s 203.0.113.0/24 --dport 873 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT 
sudo iptables -A OUTPUT -p tcp --sport 873 -m conntrack --ctstate ESTABLISHED -j ACCEPT
The second command, which allows the outgoing traffic of established rsync connections, is only necessary if the OUTPUT policy is not set to ACCEPT.  

 

 

Service: Web Server  

Web servers, such as Apache and Nginx, typically listen for requests on port 80 and 443 for HTTP and HTTPS connections, respectively. If your default policy for incoming traffic is set to drop or deny, you will want to create rules that will allow your server to respond to those requests.  

Allowing All Incoming HTTP  

To allow all incoming HTTP (port 80) connections run these commands:  

sudo iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT 
sudo iptables -A OUTPUT -p tcp --sport 80 -m conntrack --ctstate ESTABLISHED -j ACCEPT
The second command, which allows the outgoing traffic of established HTTP connections, is only necessary if the OUTPUT policy is not set to ACCEPT.  

 

Allowing All Incoming HTTPS  

To allow all incoming HTTPS (port 443) connections run these commands:  

sudo iptables -A INPUT -p tcp --dport 443 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT 
sudo iptables -A OUTPUT -p tcp --sport 443 -m conntrack --ctstate ESTABLISHED -j ACCEPT
The second command, which allows the outgoing traffic of established HTTP connections, is only necessary if the OUTPUT policy is not set to ACCEPT.  

 

Allowing All Incoming HTTP and HTTPS  

If you want to allow both HTTP and HTTPS traffic, you can use the multiport module to create a rule that allows both ports. To allow all incoming HTTP and HTTPS (port 443) connections run these commands:  

 

sudo iptables -A INPUT -p tcp -m multiport --dports 80,443 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT 
sudo iptables -A OUTPUT -p tcp -m multiport --dports 80,443 -m conntrack --ctstate ESTABLISHED -j ACCEPT
The second command, which allows the outgoing traffic of established HTTP and HTTPS connections, is only necessary if the OUTPUT policy is not set to ACCEPT.  

 

Service: MySQL  

MySQL listens for client connections on port 3306. If your MySQL database server is being used by a client on a remote server, you need to be sure to allow that traffic.  

Allowing MySQL from Specific IP Address or Subnet  

To allow incoming MySQL connections from a specific IP address or subnet, specify the source. For example, if you want to allow the entire 203.0.113.0/24 subnet, run these commands:  

sudo iptables -A INPUT -p tcp -s 203.0.113.0/24 --dport 3306 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT 
sudo iptables -A OUTPUT -p tcp --sport 3306 -m conntrack --ctstate ESTABLISHED -j ACCEPT 
The second command, which allows the outgoing traffic of established MySQL connections, is only necessary if the OUTPUT policy is not set to ACCEPT.  

 

Allowing MySQL to Specific Network Interface  

To allow MySQL connections to a specific network interface—say you have a private network interface eth1, for example—use these commands:  

sudo iptables -A INPUT -i eth1 -p tcp --dport 3306 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT 
sudo iptables -A OUTPUT -o eth1 -p tcp --sport 3306 -m conntrack --ctstate ESTABLISHED -j ACCEPT 
The second command, which allows the outgoing traffic of established MySQL connections, is only necessary if the OUTPUT policy is not set to ACCEPT.  

 

Service: PostgreSQL  

PostgreSQL listens for client connections on port 5432. If your PostgreSQL database server is being used by a client on a remote server, you need to be sure to allow that traffic.  

PostgreSQL from Specific IP Address or Subnet  

To allow incoming PostgreSQL connections from a specific IP address or subnet, specify the source. For example, if you want to allow the entire 203.0.113.0/24 subnet, run these commands:  

sudo iptables -A INPUT -p tcp -s 203.0.113.0/24 --dport 5432 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT 
sudo iptables -A OUTPUT -p tcp --sport 5432 -m conntrack --ctstate ESTABLISHED -j ACCEPT
The second command, which allows the outgoing traffic of established PostgreSQL connections, is only necessary if the OUTPUT policy is not set to ACCEPT.  

 

Allowing PostgreSQL to Specific Network Interface  

To allow PostgreSQL connections to a specific network interface—say you have a private network interface eth1, for example—use these commands:  

sudo iptables -A INPUT -i eth1 -p tcp --dport 5432 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT 
sudo iptables -A OUTPUT -o eth1 -p tcp --sport 5432 -m conntrack --ctstate ESTABLISHED -j ACCEPT
The second command, which allows the outgoing traffic of established PostgreSQL connections, is only necessary if the OUTPUT policy is not set to ACCEPT.  

 

Service: Mail  

Mail servers, such as Sendmail and Postfix, listen on a variety of ports depending on the protocols being used for mail delivery. If you are running a mail server, determine which protocols you are using and allow the appropriate types of traffic. We will also show you how to create a rule to block outgoing SMTP mail.  

Blocking Outgoing SMTP Mail  

If your server shouldn’t be sending outgoing mail, you may want to block that kind of traffic. To block outgoing SMTP mail, which uses port 25, run this command:  

sudo iptables -A OUTPUT -p tcp --dport 25 -j REJECT 
This configures iptables to reject all outgoing traffic on port 25. If you need to reject a different service by its port number, instead of port 25, substitute that port number for the 25 above.  

Allowing All Incoming SMTP  

To allow your server to respond to SMTP connections on port 25, run these commands:  

sudo iptables -A INPUT -p tcp --dport 25 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT 
sudo iptables -A OUTPUT -p tcp --sport 25 -m conntrack --ctstate ESTABLISHED -j ACCEPT
The second command, which allows the outgoing traffic of established SMTP connections, is only necessary if the OUTPUT policy is not set to ACCEPT.  

 

Allowing All Incoming IMAP  

To allow your server to respond to IMAP connections, port 143, run these commands:  

sudo iptables -A INPUT -p tcp --dport 143 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT 
sudo iptables -A OUTPUT -p tcp --sport 143 -m conntrack --ctstate ESTABLISHED -j ACCEPT
The second command, which allows the outgoing traffic of established IMAP connections, is only necessary if the OUTPUT policy is not set to ACCEPT.  

 

Allowing All Incoming IMAPS  

To allow your server to respond to IMAPS connections, port 993, run these commands:  

sudo iptables -A INPUT -p tcp --dport 993 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT 
sudo iptables -A OUTPUT -p tcp --sport 993 -m conntrack --ctstate ESTABLISHED -j ACCEPT 
The second command, which allows the outgoing traffic of established IMAPS connections, is only necessary if the OUTPUT policy is not set to ACCEPT.  

 

 

Allowing All Incoming POP3  

To allow your server to respond to POP3 connections, port 110, run these commands:  

sudo iptables -A INPUT -p tcp --dport 110 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT 
sudo iptables -A OUTPUT -p tcp --sport 110 -m conntrack --ctstate ESTABLISHED -j ACCEPT
The second command, which allows the outgoing traffic of established POP3 connections, is only necessary if the OUTPUT policy is not set to ACCEPT.  

 

Allowing All Incoming POP3S  

To allow your server to respond to POP3S connections, port 995, run these commands:  

sudo iptables -A INPUT -p tcp --dport 995 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT 
sudo iptables -A OUTPUT -p tcp --sport 995 -m conntrack --ctstate ESTABLISHED -j ACCEPT
The second command, which allows the outgoing traffic of established POP3S connections, is only necessary if the OUTPUT policy is not set to ACCEPT.  
