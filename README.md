#Vagrant for Dummies

#Pre-requirements
To start using Vagrant go to
  - http://www.vagrantup.com/downloads
and download the installation file according to your OS and then follow the simple wizard to getting started

##Concepts

  - Box: it's a package, a representation of the virtual machine on which has been installed a specific OS for a Provider

  - Provider: it's a block of code that create and manage the virtual machine. It's the one who serve the VM, usually virtual box

  - Provisioner: don't be confused by Provider, Provisioner it's the one who runs tasks using the VM served by the Provider.
  Provisioner is used to configuring the virtual server and installing the packages for your application. The most famous are Puppet, Chef e Ansible.

#Vagrantfile
    - Reference http://docs.vagrantup.com/v2/vagrantfile/index.html
The basic configuration is contained in this file and is generated in the root of your project executing the command:

    - vagrant init

In this file you can choose:

    - box used as a basis (and where download it), a package with an operating system running on your virtual machine
        config.vm.box = "precise64"
        config.vm.box_url = "http://files.vagrantup.com/precise64.box"

    - port forwarding configuration
        config.vm.network "forwarded_port", guest: 80, host: 8080

    - provider setup
        config.vm.provider "virtualbox" do |vb|

###Initial useful commands

    - vagrant Up: execute the commands of the vagrant file from next boot which will download and install the Box executing all the tasks

    - vagrant ssh: with this command you will be able to connect to your VM via ssh

#Provisioning
##With shell
The shell provisioner allows you to execute a shell script inside the vagrant box, as root.
You can find an example in my repo

    - https://github.com/andreafspeziale/vagrant_test

Be sure to use in the vagrant file the shell as provisioner

    - config.vm.provision :shell, :path => "test.sh"

then try to run.

    - vagrant up

As you will see, the script will successfully executed in the end.

If your vagrant is already up, you can run a

    - vagrant reload

to "restart" it or a

    - vagrant provision

if you only want to re-run the provisioners tasks.

For turning the machine off, run

    - vagrant halt

 or if you want to start from scratch

    - vagrant destroy

this will destroy any changes made to the base box.

##With Ansible
Why? Because Ansible is just SSH.

Chef and Puppet both have dependencies that must be installed on the server before you can use them, Ansible does not. It runs on your machine and uses SSH to connect to the servers and run the required commands.

    config.vm.provision :ansible do |ansible|
        ansible.playbook = "playbook.yml"
    end

Ansible works by running a series of Tasks on your server.
Think of a Task as a single Bash command. You then have what is called a Playbook

##Playbook
A Playbook tells Ansible what Tasks you want to run on your server. Each Task is run using an Anisble Module. A Module is a built-in way to do something, like access Yum, create a user and so on. That will become clearer later.

Your Playbook should be a YAML list. The list should contain a list of hosts (servers) you want your Playbook to run on, as a single Playbook can control multiple hosts, and then a list of Tasks you want to run on that host.

We're using Vagrant and only have one host for now, so we can just set the value to all, which is a magic value that says: "run these tasks on all servers you know about". Then we tell Ansible that these tasks will require sudo and finally add our tasks: key to begin specifying our Task list.

##Templates and Files
You will probably want to include related tasks in your new include files.
As an example, we'll update your Apache DocumentRoot setting to point at /vagrant rather than /vat/www/html, so you can serve your local files.

The simplest wat is to include your Virtual Host as part of your Ansible Playbook, either in the form of a Template or a File.

In order to create our Virtual Host, we need to follow these four steps:

    - Copy the new Virtual Host (with the DocumentRoot set to /vagrant) to /etc/apache2/sites-available

    - Disable the 000-Default Virtual Host

    - Enable our new Virtual Host

    - Reload Apache config.

So we have:

    - created a templates folder

    - created a file called virtual-hosts.conf.j2 in which we paste basic virtual host syntax with a DocumentRoot as a variable jinja2

because of the way Apache works, you need to add a new config section for the /vagrant directory, otherwise you'll get 403 Forbidden errors.

Now that we saved the file, open up tasks/apache.yml, we'll need to add a new Task to copy this Template to the server.

You don't need to include the templates directory in the src value, Ansible always assumes Templates will be in a templates directory, relative to the current context (which is important for when we look at Roles later).
If it can't find the file in the templates directory, it will check again in the root before giving up.

The final thing we need to do to copy our template across is to tell Ansible what to use for the value of document_root when it's copying the Template in to place.
Again, there are a few ways to do this, but the simplest is to add a vars: key to your playbook.yml.

The vars: key can live in any position, but convention dictates it comes somewhere before tasks: in your Playbook.  

Test it simply:

    - vagrant provision

    - cd /etc/apache2/sites-available and you will find out vagrant configuration

##The File Module
Ubuntu's configuration convention for Apache Virtual Hosts is to put the contents of all possible Virtual Hosts in /etc/apache2/sites-available and then use the a2ensite and a2dissite to create and remove symlinks for each active Virutal Host in the /etc/apache2/sites-enabled directory.

We need to add two tasks to our tasks/apache.yml file, one to remove the existing symlink, and one to create our new one

##Handlers
Telling a service to restart or reload itself manually after every configuration change can be a real pain. Fortunately, Ansible has a way of automating that process for you, and that's by using Handlers.

Handlers are a list of Tasks with a name assigned that you can call by name after another Task has run. They won't run unless another Task triggers them.

Now we tell Ansible to run this new Handler whenever we change the apache config.
To do this we add a list of Handlers to notify when a Task is complete

#Roles
What happens if we want to write a generic Playbook or set of Tasks and then re-use them?

What happens when we want configure more than just a couple of modules?

Our playbook.yml is going to become full of unrelated Handlers and variables and will get difficult to maintain.

Ansible can solve this problem with Roles. A Role is a group of Tasks, Handlers, Variables, Templates, Files, and so on all related to a single purpose.
You could create specific "apache", "MySQL", and "PHP" Roles, or a generic high-level "webserver" Role. It doesn't matter.

We'll try to create two specific roles:

    - A Webserver Role (apache + PHP)
    - A Database Role (MySQL)

##Main Task File
You'll notice that with our list of Roles we're just specifying the name of the Role, which is just the name of the directory that contains the Role.
Ansible doesn't know which Tasks to run inside that Role though, as a Role can contain multiple Task files.
The secret here is that Ansible will always look for a Task file called main.yml inside the tasks directory of a Role.

##Pre & Post Tasks
Before we try this out, there's one more thing we need to do.
Ansible runs your Roles before your Tasks, so our packages will be installed before we've updated the apt cache.
To fix this, we need to rename our 'tasks': key in playbook.yml to 'pre_tasks'.

#More Interesting Stuff

##Interesting Tips about forwarding
##Run custom bash script on vagrant up command  
##Port forwarding
##Host-manager
