lxc-amigo
======
*Now with 30% fewer NSA back doors!*

lxc-amigo is a short ruby script that adds a few helpful commands for use with lxc containers:

    lxc-amigo <command> <container> [options]

* `ip` - get the ip of the running container.
* `ssh` - ssh into the specified container
* `mkvhost` - create an nginx forwarding vhost on the host instance that
forwards all traffic into the specified container.
* `rmvhost` - remove either a single vhost for a container, or all
vhosts associated with the specified container
* `top` Show current memory and cpu usage of all containers

**note**: the container name is actually the container host name. If you create the container with lxc, then the 
host name and the container name will be the same. If you use vagrant-lxc, however, the hostname is set 
via config.vm.hostname (you should be sure you set this explicitly if you use vagrant lxc).

##installation
It's one-script, baby. Copy lx-amigo to /usr/local/bin. Currently runs
with ruby 1.8.7 and 1.9.x. 2.0 has not been tested, but probably works.

##examples

####get container ip

    $ lxc-amigo ip foo
    10.0.3.115

####ssh in to container

    $ lxc-amigo ssh ubuntu@foo  
    ubuntu@10.0.3.57's password:
    Welcome to Ubuntu 12.04.3 LTS (GNU/Linux 3.2.0-36-virtual x86_64)
    ubuntu@foo:~$    

####Create an nginx forwarding host.
Container should be running (as we need the ip)

+ -s (--server) is the public vhost name

+ -h (--host) is passed in the http Host: header to the container    

```    
    $ lxc-amigo mkvhost foo -s container-3.example.com -h internal-server.example.com
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
```

####Remove a single vhost
    
    $ lxc-amigo rmvhost  foo -s container-3.example.com
    Removing container-3.example.com
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
    
####Remove all vhosts associated with a container
    
    $ lxc-amigo rmvhost  foo
    Removing container-1.example.com
    Removing container-2.example.com
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
    
####Show container resource usage
Shows rss, private dirty and %cpu (relative to host)

    $ lxc-amigo top
                                   name          rss       pdirty  cpu%
                                     foo      348824k      270980k   0.2
                                    foo2      555144k      416716k   0.7

##Getting startid with lxc

Fire up an ubuntu 12.04 vm and be sure lxc and tools are installed. `apt-get install lxc`

You may create a new container with lxc-create using one of the templates that come with lxc. For example:

    lxc-create -n base -t ubuntu
    
That creates a fresh ubuntu (as of this writing) 12.04 install from the `ubuntu` template in /usr/lib/lxc/templates. 
There are a few templates to choose from, and most OpenVZ templates are
also said to work. You can cherry this up if you are so inclined with whatever you'd like in
your base image.

You may now clone new containers from this base image. So:

    $ lxc-clone -o base -n my-container # Create new container from base This takes a while
    
    # starts my-container and drops you into console. Creds: ubuntu/ubuntu
    # note that there is no way to get out of this console (just like a real console)
    # short of logging in and shutting down, or issuing an lxc-shutdown in another session
    $ lxc-start -n my-container 
    
    # Better, start the container as a daemon, and ssh in
    $ lxc-start -d -n my-container
    $ lxc-amigo ssh ubuntu@my-container
        <enter password and do stuff. ubuntu user has passwordless sudo>
        
    # Forward http traffic via vhost to it if you have a webserver running
    $ lxc-amigo mkvhost my-container -s 'my-container.example.com' -h 'whatever'
    
    # When you are done with the container and wish to shut it down, logout and
    
    $ lxc-shutdown -n my-container
    
    # any changes you made to the container are still on disk. The container filesystem is at 
    # /var/lib/lxc/my-container/root-fs
    
    # You should probably kill off any vhosts on the host pointing to your container
    $ lxc-amigo rmvhosts my-container
    
You can also start a container with a specific command, and the container will exit when that command is 
done. When a command is ommitted, /sbin/init is started. But you can run any program:

    $ lxc-start -n my-container -- '/bin/ls'
    bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  selinux  srv  sys  tmp  usr  var
    
The container then exits immediately. 

You can mount a directory from the host by adding an entry to the containers fstab file
(in `/var/lib/lxc/<container>/fstab`). For example:

    /media/psf  /var/lib/lxc/test/rootfs/media/psf  none bind 0 0

mounts the Parallels host share into the test container. Note the full pathname to the mount point in `test`. 

For more info specific to ubuntu, see https://help.ubuntu.com/lts/serverguide/lxc.html

##vagrant-lxc
vagrant-lxc adds lxc support to vagrant and is a great way to to build
and provision containers. See https://github.com/fgrehm/vagrant-lxc
for more information.
 
