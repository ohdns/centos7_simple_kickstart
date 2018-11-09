# CentOS 7 Simple Kickstart

Loosely maintained.  My primary focus will be moving services to docker images and using smaller and lighter systemd-less
distros like Alpine Linux [0] and CoreOS [1].

[0] https://alpinelinux.org/
[1] https://coreos.com/

# Goals:
<p>
<ul>
<li>Create an extremely simple and trimmed down CentOS 7 kickstart environment that produces small CentOS 7 images.</li>

<li>Use nearly the same method that a person might use to create this on bare metal servers in a data-center.<br />One could even build servers in a data-center off their laptop if they add a bridged or NAT interface to the Kickstart VM.</li>

<li>Use <b>only</b> what Apple provides us on the Mac, plus VirtualBox if it is not already installed.</li>

<li>Avoid requiring sudo or root on the laptop.</li>

<li>Avoid vagrant as it is not repeatable in the data-center.</li>

<li>Learn how kickstart works and what is required to build a simple CentOS 7 machine.</li>
</ul>
</p>
___

# Notational Conventions

<p>The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in <b>RFC-2119</b>.</p>
<br />
<p>The key words "MUST (BUT WE KNOW YOU WON'T)", "SHOULD CONSIDER", and "REALLY SHOULD NOT" are to be interpreted as described in <b>RFC-6919</b>.</p>

___

# Requirements:
<p>
<ul>
<li>Mac Laptop with VirtualBox installed.</li>

<li>Access to the internet on TCP ports 80 (http), 443 (https) and 873 (rsync)</li>

<li>About 100 GB of free space on your hard drive. <i>Actual usage may be much less depending on number of VM you create.</i></li>

<li>Some type of command line interface.  Most folks use <b>Terminal</b> on Mac.</li>
</ul>
</p>
___

# Steps
<p>
In these steps, we will be syncing the public CentOS and Fedora EPEL repos to a staging location on your laptop, then starting a local low privileged instance of rsync and apache on your private vboxnet interface.<br /><br />
From there, the kickstart process will read your kickstart configuration files and build the first VM, using the files from your laptop. After the first VM is built, all future VM's will then be built directly from your "kickstart server" VM from step2.<br /><br />

<ul>
<li><b><font color="#960000">Install</b></font> VirtualBox if you have not done so already.  See virtualbox.org</li>

<li><b>Clone</b> this repo.  See https://github.com/ for instructions on how to clone repos.</li>
<code>
mkdir -p ~/build/centos7_simple_kickstart/scripts
</code>
<br />
<code>
git clone https://github.com/ohdns/centos7_simple_kickstart.git
</code>
<br />
<code>
rsync -av centos7_simple_kickstart/. ~/build/centos7_simple_kickstart/scripts/.
</code>
<br /><br />
<b>This is so that you have a working copy you can edit. &nbsp; Please do feel free to fork this.</b><br /><br />

<li><i>OPTIONAL:</i>
 Edit the .cfg files in ~/build/centos7_simple_kickstart/scripts and replace the SHA512 hashes with your own.<br />
I would show how to generate a SHA512 shadow hash, but each OS and each version of Python behaves dramatically different in this area.<br /><br />
The <b>default password</b> for <b>ohadmin</b> and <b>root</b> will be <b><i><u>centos7</u></i></b><br />
The <i>ohadmin</i> account will be used for automation; and <i>temporarily</i>, the way you ssh to your VM for manual changes or container deployments.
<br /><br /><br />
<b>It is assumed that you will change all passwords and ssh keys before doing something risky, such as bridging your kickstart VM onto a non private network</b>
<br /><br /><br />

<li>
 <b>Step 1: Execute</b> <code>~/build/centos7_simple_kickstart/scripts/oh repo_sync</code>
<br /><br />
This step will rsync the CentOS and Fedora EPEL repos to your laptop in a staging location.
<br /><br />
This will run peridoically from the kickstart VM, pulling and mirroring the updated repos from your laptop, so <b>run this step once per week.</b>
<br /><br />
</li>

<li>
 <b>Step 2: Execute</b> <code>~/build/centos7_simple_kickstart/scripts/oh ssh_config</code>
<br /><br />
This step will create a ssh key pair for each new VM that this script supports.  <b>You should only have to run this step once.</b>
<br /><br />
</li>

<li>
 <b>Step 3: Execute</b> <code>~/build/centos7_simple_kickstart/scripts/oh build_ks</code>
<br /><br />
This step will perform the following:<br />
Start up a local apache instance on 192.168.120.1 on a new private network of 192.168.120.0/24<br />
Start up a local rsync daemon on a high port on 192.168.120.1<br />
Kickstart your <i>new kickstart VM server</i>.  <b>You should only have to run this step once.</b>
<br /><br />
<b>When the first VM starts</b>, you should see a CentOS ISO install screen.<br />
<b>At this point</b>, hit the TAB key, backspace over <i>quiet</i> and type:<br /><br />
<code>cmdline ip=192.168.120.10 netmask=255.255.255.0 ks=http://192.168.120.1:8888/c7_server.cfg nicdelay=20</code><br /><br />
This will build the first VM (the kickstart server role) and will rsync the Yum repos to itself, pulling from your laptop. Get a cup of coffee or tea while this runs.<br /><br />
Once this step completes, you should be able (from your laptop terminal) to type: <code>ssh ohkickstart</code><br />
<br /><br />
</li>

<h3>Now let us deploy some server roles using our shiny new Kickstart VM</h3>

<li>When the Kickstart VM completes building:<br /><br />
 <b>Step 4: (Optional) Execute</b> <code>~/build/centos7_simple_kickstart/scripts/oh build_docker</code> to spin up a docker VM.<br /><br />Unless you change the <b>oh</b> script functions, the VM's will use up to 1.5 GB of ram each. Most laptops do not have more than 16 GB of ram. By all means, fiddle with the memory allocation to see what you can create. CentOS 7 can run in a small amount of memory and with a tiny disk, but it wants some disk space to stage temporary files.<br /><br />
When this step completes, you should be able to type: <code>ssh ohdocker</code> from your laptop terminal.<br /><br />
</li>

<li>Build a DevOps Workstation / Laptop Image<br />
 <b>Step 5: (Optional) Execute</b> <code>~/build/centos7_simple_kickstart/scripts/oh build_devops</code>
<br /><br />
This step will create a DevOps workstation with many of the tools and services one might need such as mcollective, ruby, perl, python, various compilers and libraries, etc...
<br /><br />
</li>
</ul>
</p>
___

# Some Challenges For You!

<p>
<ul>
<li>Keep SELinux enabled, no matter what you plan to install.<br />
Instead of disabling SELinux, read up on setting booleans (<i>setsebool -P boolean or getsebool -a</i>), creating rules, setting file and directory contexts (semanage fcontext)<br />
Confine as many applications as you can, to the least amount of privilages required to run the application.<br />
Use audit2allow, audit2why or grep through /var/log/audit/audit.log to see why something was denied.<br />
<b>As a last resort</b>, set a user or process to <i>unconfined</i> or <i>permissive</i> instead of disabling SELinux.</li>

<li>Prefer a chroot restricted SFTP over unrestricted SSH trusts when feasible.<br />
You can accomplish the same behavior of rsync using SFTP Chroot + LFTP and it's mirror subsystem, without having to expose your entire system or provide shells to people. It is faster than rsync and more secure.<br />
</li>

<li>For custom applications, clearly define:<br />
where the application binaries should be installed. <i>i.e. /opt/application_name/{etc,lib,include,bin,sbin}</i><br />
where the application logs should reside. <i>i.e. /data/application_name/{var,logs}</i><br />
where transitory data should reside. <i>i.e. /data/application_name/{data,db}</i><br />
or better, <b>use containers!</b> to keep the OS pristine and easy to patch.</li>

<li>Configure your applications, users, and group posix permissions to allow the right folks to view logs, restart services, or otherwise manage their job role without having to elevate privs to root.  If non sysadmins are needing root or sudo to get their job done, then the sysadmin has more work to do, sorry.  Get a sysadmin, a developer, a devops person into a room and sort it out.  You will save thousands of hours or work for yourselves.</li>

<li>Aside from the Automation account, avoid SSH key trusts when you can. Proper automation is derrived from codified instructions that execute on orchestration servers from within data-center, not from a laptop.</li>

<li>Avoid enabling root login via ssh.  In fact, avoid logging into any VM's if you can.<br />
The base configuration belongs in kickstart and customization needs to be done in a configuration management and orchestration system.<br />
All services need to be initialized by a proper startup script when the server OS starts up.</li>

<li>Avoid using sudo. If you find yourself using <code>sudo su -</code> or <code>sudo -s</code> then you really just need the root password.
 <b>Sudo is not a security tool.</b> It is designed to allow a sysadmin to delegate commands <i>as a user</i>, to a set of users.
 If you can get a root shell through sudo, then it is misconfigured.  
 Sudo is potentially a critical security risk if your users have un-restricted sudo and they get phished. This is equally a risk in both a production customer facing environment and a development or continuous integration environment, as both ultimately touch customer data.<br /><br />Create yourself a counter. &nbsp; Every time you type sudo, subtract 10 engineering points.</li>

</ul>
</p>
<br />
___


# Known Issues and Limitations / TO-DO's:
<p>
<ul>
<li>LIMITATION and OPTIONAL: To use NTP from the kickstart server to our laptop, we have to break our own rule one time and use either root or sudo to modify /private/etc/ntp-restrict.conf on our laptop to allow a query.<br />
<code>sudo echo "restrict 192.168.120.0/24" >> /private/etc/ntp-restrict.conf ; sudo pkill -HUP ntpd</code><br /></li>

<li>SUB-OPTIMAL: There is one manual step to create the DHCP/PXE/Yum/Kickstart server.  This is <i>probably</i> ok, since we should not be doing this often.</li>

<li>SUB-OPTIMAL: This method currently lacks end-to-end validation of the RPM GPG signatures.  (Work In Progress, Contacting CentOS Team.)</li>

<li>TO-DO: <s>You will need to manually add your SSH public key in the ~/build/centos7_simple_kickstart/scripts/c7*cfg</s> <b>Completed and Rearchitected.</b></li>

<li>TO-DO: <s>Merge all the steps into 1 script as functions</s>. <b>Completed.</b></li>

<li>TO-DO: <s>Add cron entry to the _kickstart_ VM to periodically sync its mirror of CentOS and EPEL from the Laptop.</s> <b>Completed.</b></li>

<li>Known Issue: The <b>oh</b> script does not currently pass ShellCheck best practices. WIP</li>

<li>WORKS AS DESIGNED: There is currently a bit of customization in the kickstart files.  <b>This is on purpose.</b>  We start with functional customizations in kickstart, so that <b>anyone</b> can easily figure out what needs to be customized in Ansible, Puppet, Chef, cfengine or whatever their configuration management flavor preference may be.</li>

<li>WORKS AS DESIGNED: I renice this script to avoid blocking any work you are doing. This means if your laptop is under a heavy load, this kickstart build process may go very slow by design.</li>

<li>WORKS AS DESIGNED: Since we are not using Vagrant, you would have to script startup/shutdown yourself.<br /><br />

<b>Examples:</b><br /><br />
<code>PATH=${PATH}:/Applications/VirtualBox.app/Contents/MacOS;export PATH</code><br /><br />
Clean / Graceful Power Off:<br />
<code>VBoxManage controlvm c7_docker acpipowerbutton</code><br /><br />
Power On:<br />
<code>VBoxManage startvm c7_docker</code><br /><br />
Power On Headless (No GUI / console):<br />
<code>VBoxManage startvm --type headless c7_docker</code><br /><br />
</li>
</ul>
<br />If you use VirtualBox a bit from the command line, then you may with to update the PATH in your ~/.bash_profile<br /><br />
</p>


___


License: WTFPL  see http://www.wtfpl.net/txt/copying/


___


<p><b><br />Disclaimer: This software repository contains scripts that are for educational purposes only.  This repo contains default passwords that must not be used anywhere beyond VirtualBox on your laptop for educational purposes only. DO NOT use this to deploy a production environment unless you have properly changed all defaults, changed settings to reflect that which is approved for your environment and have properly tested this in a lab and staging area that matches your live environments.  The author of these scripts assumes no responsibility for damages to persons or property.  Do not bridge any VM's to a network outside of localnet without first understanding the potential consequences and having the appropriate personnel in your organization validate and accept the risks.<br /><br /></b>TL;DR - <i>Get someone else to sign off and accept the risk for using someone elses scripts in your environment.</i><br /><br /></p>

___


<i>20180608</i>


