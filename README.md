# osx_docker
Ubuntu 14.04 Vagrant image with Ansible setup for OSX, because boot2docker is just too limited.  
  
This setups up a VM with a static IP running docker, configures and mounts /Users with NFS for full user support, installs docker for OSX locally and sets the `DOCKER_HOST` environment variable so you can access docker from the OSX cli. 

**Why NFS?** VirtualBox shares set the permissions on the share to your uid. That means if docker wants to run a command and write to storage on the /Users share as something other than your uid or 0 (root) it will fail. This was a big pain point for me when trying to use standard packaging for things like RabbitMQ, Redis and others. 

**Why not boot2docker** boot2docker is soooo stripped down that I find provisioning any customizations after its built and installed a hair-pulling experience. Frankly the 100MB of Ram and 400MB of disk its saving is just not worth the hassle. 

## Requirements
 * xcode cli tools `xcode-select --install`
 * Homebrew http://brew.sh/
 * Homebrew Cask `brew install caskroom/cask/brew-cask` 
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
 * Set `DOCKER_HOST` in ~/.profile, and ~/.bash_profile

The Vagrant VM provisions the vagrant.yml file on startup.
 * Create /Users and mount nfs share.
 * Install aufs support (devicemapper fails randomly during `docker build`)
 * Install Docker from docker.com apt repo.
 * Configure docker to allow tcp connections.

## Using Docker
 * Make sure the Vagrant VM is started.
 * Run docker commands from the osx console.

#### Starting Vagrant
`cd <repo>` and run `vagrant up` to start the vm. 

#### Auto-Start Docker Containers
Run docker contaners with `--restart=always` option to have continers start on `vargrant up`

#### Volume Tricks
Theses are some of the ways I take advantage of /Users shared via NFS. 
 * Use `--volume /Users/jgreat/.docker/my_continaer_name:/data` to save persistent data in your home directory.
 * Use `--volume /Users/jgreat/MyGitHubProject:/app` and over mount your application dir so you can develop live in your docker.

#### Accessing Containers
 * Services "published" on the vm running docker can be reached on 192.168.59.103.
 * Set entries in /etc/hosts: `192.168.59.103  www-dev.jgreat.me`

#### Troubleshooting
  * If you get `Cannot connect to the Docker daemon.`
    * Make sure the VM is runing
    * check to see that `DOCKER_HOST` is set correctly  
To check:
```
$ echo $DOCKER_HOST
tcp://192.168.59.103:2375
```
To set if incorrect:
```
$ export DOCKER_HOST=tcp://192.168.59.103:2375
```

