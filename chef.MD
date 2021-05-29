# Download & install Chef and create Cookbook, Recipes on the EC2
```
	wget https://packages.chef.io/files/stable/chef-workstation/21.4.365/el/8/chef-workstation-21.4.365-1.el7.x86_64.rpm
	yum install chef-workstation-21.4.365-1.el7.x86_64.rpm -y
```
**Q. How to create a folder to store the cookbooks and go in it?**
```
	mkdir cookbooks
	cd cookbooks/
```
Q. How to create a cook book apache under the folder cookbooks?
```
	chef generate cookbook apache 
```
> ( Note: press y to complete the process.)
```
	cd apache
```


Q. How to Install package tree to view tree of directories?
```
	yum install tree -y
	tree
```
Q. How to create a recipe under the apache cookbook?
```
	chef generate recipe test1	
	cd ..
	vi apache/recipes/test1.rb
```
Note: Add the code blow into the file followed by pressing “I” to insert the code.
```
file '/myfile' do
content 'Welcome to Cpanel Guru' 
action :create
end
```
>	Note: enter+save the file esc+:wq  (to save the file.)

Q. How to check the syntax of the recipe?
```
chef  exec ruby -c apache/recipes/test1.rb
```
>	Syntax OK

How to run the chef client?
```
	chef-client -zr “recipe[apache::test1]” 
	cat /myfile(xyz)
```
>	(also try ls /)	(to check inside the file)

How to install Apache server using cookbook recipe?
```
cd / cookbooks
chef generate cookbook apache 
cd apache
chef generate recipe apache 
cd ..
vi apache/recipes/apache.rb
```
> Note: Add the code blow into the file followed by pressing “I” to insert the code.
```
package 'httpd' do 
action :install
end

file '/var/www/html/index.html' do 
content 'Welcome to CpanelGuru' 
action :create
end

service 'httpd' do 
action [:enable, :start]
end	
```
Note: save the file esc+:wq [to save the file]
```
	chef-client -zr “recipe[Apache::Apache]”
```
Now browse public IP address in web browser to see content on apache website.

(______________________________________________________________________________________________________)
#	ATTRIBUTES:

Q. What is Attributes?
Attributes is a key value pair which represent a specific detail about node.

Q. Who use attributes? 
Chef client

Q. Why attributes are used by chef client? 
To determine below state of the node (client servers)
	current state of node?
	what was the state of the node at the end of previous chef client run?
	What should be the state of the node at the end of current chef client will run?


Types:	Priority
1.	Default	1st maximum
2.	Force-default	2nd more
3.	Normal	3rd may be
4.	Override	4th less
5.	Force override	5th very less
6.	Automatic	6th minimum

Who defines Attributes?
Ans: (Node, Cookbooks, Roles, Environment)	**(attribute defines by Ohai have highest priority)
```
sudo su # ohai
ohai ipaddress
ohai memory/total # ohai cpu/0/mhz
ls
cd cookbooks
cd Apache
chef generate recipe myrecipe # cd ..
vi apache/recipes/myrecipe.rb
```
>	Note: Add the code blow into the file followed by pressing “I” to insert the code.
```

file '/nodeinfo' do 
content "This is to get Attributes 
HOSTNAME: #{node['hostname']} 
IPADDRESS: #{node['ipaddress']} 
CPU: #{node['cpu']['0']['mhz']} 
MEMORY: #{node['memory']['total']}" 
owner 'root' 
group 'root' 
action :create end
```
>	Save the file esc+:wq
```
	chef exec ruby -c apache/recipes/myrecipe.rb
	chef-client -zr “recipe[apache::myrecipe]”	
```
(call the client)

SEE OUTPUT ATTRIBUTES
Insert Linux commands
```
vi apache/recipes/second-recipe.rb
 ``` 
> I + Enter then <paste code>
> [Note: ----> EOH = End of here/hunk (now can write non ruby)]

```
execute 'apache_configtest nd create a file and folder' do
 command <<-EOH
'/usr/sbin/apachectl configtest'
mkdir '/test'
touch '/bfile'
EOH
end
```
Enter+Save the file esc+:wq

Create user
```
vi apache/recipes/second-recipe.rb
```
> Add code below
```
user “amol” do
action :create
end
```
>	Enter+Save the file esc+:wq	(now run the recipe)
```
	chef-client -zr “recipe[apache::Second-recipe]”
```
Create group
```
vi apache/recipes/Second-recipe.rb
```
>	Note: Add the code blow into the file followed by pressing “I” to insert the code.
```
group "isha" do
action :create
members 'amol'
append true
end
```
Enter+Save the file esc+:wq	(now check the recipe)
```
Chef exec ruby -c apache/recipes/second-recipe.rb
```
>	 syntax OK	(now run the recipe)
```
chef-client -zr “recipe[apache::second-recipe]”
 cat /etc/group
 ```
>	(also try ls /)	(to check the group)

Directly cerate a group and file for it.
```
user "guru"
file "cpanelguru"
```
Save the file and run chef-client -zr

RUNLIST: To run the recipe in a sequence order that we mention in a run list. With this process we can run multiple recipes but the condition is, they must be only one recipe from one cookbook.
***(chef client calling default recipes from apache & Apache together) ***
```

chef-client -zr "recipe[apache::default],recipe[best::default]" 
```
Include Recipe: To call recipes/recipe from another recipe with in same cookbook. To run multiple recipes from same cookbook. We can run any numbers of recipes with include command but all must be from same cookbook. Here including recipes with default recipe in apache.

```
vi apache/recipes/default.rb
```
Note: Add the code blow into the file followed by pressing “I” to insert the code.
```
include_recipe "apache::apache"
include_recipe "apache::myrecipe"
include_recipe "apache::second-recipe"
```
Save the file esc+:wq	(call the chef client)
```
chef-client -zr “recipe[apache::default]”
``` 

Connect workstation to chef server to node using chef-repo, bootstrap
Chef server is going to mediator between the code and cookbooks.
Bootstrapping Attaching a node to chef server, by Bootstrap command, both workstation and node should be in same AZ. 
Two actions will be done while bootstrapping 
1. adding node to chef server 
2. installing chef package.

Chef-repo It would be the main directory inside it you have to run any commons, it is also having cookbooks).
Create chef manage account by “manage.chef.io” and download starter kit. Go to download and extract file chef-repo, after extracting we got more files inside chef-repo are (.chef ,cookbooks
,gitignore, README.md, roles)

For sending chef-repo file to Linux machine we use the software called WinSCP. Drag Chef-repo from left window and drop to right Linux window. (by ls command in you can check Chef-repo is showing in your workstation or not)

```
sudo su
ls
```
chef-repo 
chef-workstation-20.7.96-1.e17.x86_64.rpm 
cookbooks 
nodes
 ```
 cd chef-repo
 ls-a
```
. .. .chef cookbooks .gitignore README.md roles
```
cd .chef/
ls -a
```
config.rb	
cpanelguru.pem
>	(organization name is CpanelGuru)
```
 cat config.rb
```
*Inside config.rb you will get	chef_server_url	https://api.chef.io/organizations/cpanelguru

>	(to check workstation is connected with chef server ?)
```
knife ssl check  
```

Connecting to host api.chef.io:443
Successfully verified certificates from api.chef.io
 
Create Linux machine (Node1) same AZ of workstation with new security group and new key pair name
node1-key, save Private IP for further knife bootstrap commands.	(SSH & HTTPs)

Attach Advance details	
```
#!/bin/bash
sudo su
yum update -y
```
With the help of WinSCP please transfer downloaded node1-key.pem to Chef-repo for bootstrap command

Now go to chef workstation and execute Bootstrap command to attach node1 to chef-server.

```
knife bootstrap 172.31.29.109 -–ssh-user ec2-user -–sudo -i node-2key.pem -N node1

```
>	(Y for YES/NO)


Node has been connected to server and node package has installed

(Thank you for installing Chef Infra client chef package)

```
knife node list
```
node1	?	(showing result node1 means node1 has been connected to serve
Moving and delete cookbooks in chef-repo to avoid cookbooks confusion:

How move cookbooks to chef-repo’s cookbooks
```
mv cookbooks/apache chef-repo/cookbooks
mv cookbooks/best chef-repo/cookbooks
ls -l
```
*	didn’t get any cookbook, all empty
```
cd chef-repo
ls	
```
get (cookbooks node1-key.pem README.md roles)
```
ls cookbooks/
```
apache 
chefignore 
starter 
best
	(got inside the chef-repo-cookboook)
 
It means both cookbooks have been moved to chef-repo cookboooks from  


Upload apache to chef-server:
```
knife cookbook upload apache 
```
(Cnfirm uploading with command below!)
```
knife cookbook list
```
apache

Now we will attach the recipe on node1 which we would like to run on node1, by this command
```
knife node run_list set node1 “recipe[apache::apache]”
```
Node1:
```
run_list: recipe[apache::apache]
knife node show node1
```
>	get so many info including recipes in run_list)


Now access the Node1 
```
sudo su
chef-client
```
**This chef-client implement the code (inside the recipe) on server Automatically** Now back to workstation and edit the recipe:

```
vi cookbooks/apache/recipes/apache.rb
```
“Update recipe”	Enter+Save the file esc+:wq

Upload apache to chef-server
```
knife cookbook upload apache
 ```
Now go to the node1 and call chef client # chef-client
You can see all updated content, also you can ping node1’s public IP and see change.

Now see how can we automate this process:
Go to node1
```
vi /etc/crontab
#dd code below 
* * * * * root chef-client	
```
Save the file esc+:wq	*/n (HR DAY MONTH YEAR WEEK)

With the help of this command automation will start no need to call the chef client again=2 Chef-client command execute periodically according to “*/n * * * * crontab method”

Now see full automation:
Create one more linux machine Node2	*(we also can use existing key of node1 for node2 creation)

Attach Advance details	
```
#!/bin/bash
sudo su 
yum update  -y
echo "* * * * * root chef-client" >> /etc/crontab
```
Now back to workstation and run Bootstrap command
```
knife bootstrap 172.31.10.120 -–ssh-user ec2-user -–sudo -i node-2key.pem -N node2	
```
(Y for YES/NO)

Node has been connected to server and node package has been installed

Now Attach the Recipe to node2 run_list
```
 
knife node run_list  set node2 “recipe[apache::apache]”
``` 
then browse the IP of node2 and see webpage.
How to see Delete everything from inside chef-server:

>	To see list of client present inside chef-server	To delete clients
```
knife client list
knife client delete clientname -y
```

>	To see cookbook list	To delete cookbook
```
knife cookbook list
knife cookbook delete cookbookName -y
```

>	To see Role list	To delete Role

```
knife role list	
knife cookbook delete roleName -y
```

To see Node list	To delete Node
```
nife node list
nife cookbook delete nodeName -y
```

=================================================================================
#				How to create ROLE				#
=================================================================================
```
ls
```
 .chef	
roles
```
cd roles/
ls 
```
DevOps.rb
```
vi DevOps.rb
```
>	(this is the command to create role name DevOps)
```
Name “DevOps”
Description “webserver role”
run_list “recipe[apache::apache]”
```
>	Save the file esc+:wq

Now back to chef-repo 
```
cd ..	
```
and upload the role on chef server
```
knife role from file roles/DevOps.rb
```
If you want to see the created role
```
knife role list
```
o/p:	DevOps
Now create 4 instances (1,2,3,4) by one IMA on same availability zone as of workstation with new security group sg-1 with SSH +HTTP.

Attach Advance details	
```
#!/bin/bash
sudo su
yum update -y
echo”* * * * *root chef-client”>> etc/crontab
```
Now Bootstraps the nodes 1,2,3,4 one by one
```
knife bootstrap 172.31.1.121 -–ssh-user ec2-user -–sudo -i node-1key.pem -N node1 
knife bootstrap 172.31.1.151 -–ssh-user ec2-user -–sudo -i node-1key.pem -N node2
knife bootstrap 172.31.1.131 -–ssh-user ec2-user -–sudo -i node-1key.pem -N node3
knife bootstrap 172.31.1.141 -–ssh-user ec2-user -–sudo -i node-1key.pem -N node4
```
>	Now connect these nodes to roles one by one.
```
knife node run-list set node1 “role[DevOps]”
```
Node1:
run_list:role[DevOps]
(similarly for rest 3 nodes)
```
knife node run-list set node2 “role[DevOps]” 
knife node run-list set node3 “role[DevOps]” 
knife node run-list set node4 “role[DevOps]”
```
UPLOAD cookbook to server
```
knife cookbook upload apache
```
Now we can check public IP of any node on webserver, every node will behave like server cause, now cookbook has been uploaded despite of uploading different recipes, all recipes have uploaded together inside role by cookbok.

Now we are doing changes in recipe
```
vi cookbooks/apache/recipes/apache.rb
```

Content change to “Bharat Mata ki Jay”	

Save the file esc+:wq
 
Now see if Boss need changes, said do work on another recipe (myrecipe)
```
cat cookbooks/apache/recipes/myrecipe.rb
```
Paste code update recipe and go to the role in workstation
```
vi roles/DevOps.rb
```
#ADD below lines
```
Name “DevOps”
Description “webserver role”
run_list “recipe[apache::apache]”		
run_list “recipe[apache::myrecipe]”	
```
>	(Note : update apache to myrecipe in role)
Save the file esc+:wq

for update in recipe we can create user and file by these commands below
```
user”guru”
file “samplefile”
```
now upload role to server
```
knife role from file roles/DevOps.rb
```
Again, go to the workstation
```
 vi roles/DevOps.rb
```
Add code below
```
Name “DevOps”
Description “webserver role”	(change last line only apache in role)
run_list “recipe[apache]”	
```
Save the file esc+:wq

now upload role to server
```
knife role from file roles/DevOps.rb
```
Do not mention any recipe just upload only cookbook for all recipes, will update automatically on server
``` 
knife cookbook upload apache-cookbook
``` 
Now we are adding 2 cookbooks in roles
```
vi roles/DevOps.rb
```
Add code below
```
Name “DevOps”
Description “webserver role”
run_list “recipe[apache]”,”recipe[apache]”
```
save the file esc+:wq
now upload role to server

```
knife role from file roles/DevOps.rb
```
Do not forget to upload apache on server otherwise role will not perform properly
```
knife cookbook upload apache
```
Boss need changes again but this time in test1
```
vi cookbooks/apache/recipes/test1.rb
```
Add code below
```
%W (httpd mariadb-server unzip git vim) .each do |p| Package p do
 action :install end
end	
```

save the file save the file esc+:wq
```
knife cookbook upload apache
```
Go to inside any node and search git by using command
```
which git	
```

after 1 minute execute again same command and you will see output
```
/bin/git	
```
it means working properly

|--------------------------------|------------------------------------------------------------------------------------|
|Generate cookbook               |chef generate cookbook apache                                                       |
|--------------------------------|------------------------------------------------------------------------------------|
|Chef generate			 |chef generate recipe test1		   			                      |
|--------------------------------|------------------------------------------------------------------------------------|
|Check syntax			 |chef  exec ruby -c apache/recipes/test1.rb 	    				      |
|--------------------------------|------------------------------------------------------------------------------------|
|Run chef client		 |chef-client -zr “recipe[apache::test1]”					      |
|--------------------------------|------------------------------------------------------------------------------------|
|Check ohai			 |ohai	  									      | 
|--------------------------------|------------------------------------------------------------------------------------|
|Node IP with Ohai		 |ohai ipaddress								      | 
|--------------------------------|------------------------------------------------------------------------------------|
|Node CPU with Ohai		 |ohai memory/total # ohai cpu/0/mhz		   			              | 
|--------------------------------|------------------------------------------------------------------------------------|
|Run multiple recipe		 |chef-client -zr "recipe[apache::default],recipe[best::default]"		      |	
|--------------------------------|------------------------------------------------------------------------------------|
|Include recipe	in default	 |include_recipe "apache::apache"						      |
|--------------------------------|------------------------------------------------------------------------------------|
|Check workstation is connected	 |knife ssl check								      |
|--------------------------------|------------------------------------------------------------------------------------|
|Bootstrap Connect node		 |knife bootstrap 172.31.29.109 -–ssh-user ec2-user -–sudo -i node-2key.pem -N node1  |
|--------------------------------|------------------------------------------------------------------------------------|
|Upload cookBook to the server	 |knife cookbook upload apache	  						      |
|--------------------------------|------------------------------------------------------------------------------------|
|Check cookbooks on the server	 |knife cookbook list								      | 
|--------------------------------|------------------------------------------------------------------------------------|
|Attach recipe to the node	 |knife node run_list set node1 “recipe[apache::apache]”			      |
|--------------------------------|------------------------------------------------------------------------------------|
|Check node status		 |knife node show node1								      |	
|--------------------------------|------------------------------------------------------------------------------------|
|List clients on chef-server	 |knife client list								      |
|--------------------------------|------------------------------------------------------------------------------------|
|To delete clients		 |knife client delete clientname -y						      |
|--------------------------------|------------------------------------------------------------------------------------|
|To see cookbook list		 |knife cookbook list								      |
|--------------------------------|------------------------------------------------------------------------------------|	
|To delete cookbook		 |knife cookbook delete cookbookName -y						      |
|--------------------------------|------------------------------------------------------------------------------------|
|To see Role list 		 |knife role list								      | 
|--------------------------------|------------------------------------------------------------------------------------|
|To delete Role			 |knife cookbook delete roleName -y						      |	
|--------------------------------|------------------------------------------------------------------------------------|
|To see Node list	 	 |knife node list								      |	
|--------------------------------|------------------------------------------------------------------------------------|
|To delete Node			 |knife cookbook delete nodeName -y					              |	
|--------------------------------|------------------------------------------------------------------------------------|
|Upload the role on chef server	 |knife role from file roles/DevOps.rb						      |
|--------------------------------|------------------------------------------------------------------------------------|
|Connect node to roles		 |knife node run-list set node1 “role[DevOps]”					      |	
|--------------------------------|------------------------------------------------------------------------------------|

 #You are Master in chef
