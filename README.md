# osx_docker
Ubuntu 14.04 Vagrant image with Ansible setup for OSX, because boot2docker is just too limited.  
  
This setups up a VM with a static IP running docker, configures and mounts /Users with NFS for full user support, installs docker for OSX locally and sets the `DOCKER_HOST` environment variable so you can access docker from the OSX cli. 

*Why NFS?* VirtualBox shares set the permissions on the share to your uid. That means if docker wants to run a command and write to storate on the /Users share as something other than your uid or 0 (root) it will fail. This was a big pain point for me when trying to use standard packaging for things like RabbitMQ, Redis and others. 

*Why not boot2docker* boot2docker is soooo stripped down that I find provisioning any customizations after its built and installed a hair-pulling experinece. Frankly the 100MB of Ram and 400MB of disk its saving is just not worth the hassel. 

## Requirements
 * xcode cli tools `xcode-select --install`
 * homebrew http://brew.sh/
 * Ansible `brew install ansible`

## Install
Clone this repo.  
Run the `install.yml` Ansible Playbook.  
When prompted enter your password.
```
ansible-playbook --ask-sudo-pass -i ./inventory ./install.yml
```
This playbook will: 
 * Install Docker, Vagrant, VirtualBox locally.
 * Configure NFSD to share /Users with the VM.
 * Startup the Vagrant VM.
 * Set `DOCKER_HOST` in ~/.profile

The Vagrant VM provisions the vagrant.yml file on startup.
 * Create /Users and mount nfs share.
 * Install aufs support (devicemapper fails randomly durring `docker build`)
 * Install Docker from docker.com apt repo.
 * Configure docker to allow tcp connections.

## Using Docker
 * Run docker commands from the osx console.


#### Volume Tricks
Theses are some of the ways I take advantage of /Users shared via NFS. 
 * Use `--volume /Users/jgreat/.docker/my_continaer_name:/data` to save persistant data in your home directory.
 * Use `--volume /Users/jgreat/MyGitHubProject:/app` and over mount your application dir so you can develop live in your docker.

#### Accessing Containers
 * Services "published" on the vm running docker can be reached on 192.168.59.103.
 * Set entries in /etc/hosts: `192.168.59.103  www-dev.jgreat.me`

#### Troubleshooting
 * ~/.profile sometimes isn't processed on shell startup. Take the `DOCKER_HOST` entry and put it in your .bashrc or someplace it will be processed.

 * Virtualbox _almost_ always sets the virtualbox adapter to 192.168.59.3. If it doesn't go into the VitrualBox GUI -> file -> preferences -> Network -> Host-Only Networks. Change the approprate vboxnet adapter to 192.168.59.3.
 
