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
