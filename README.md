ipam-driver
=============

Docker (libnetwork) driver for IPAM
-----------------------------------

ipam-driver is a Docker libnetwork driver that interfaces with Infoblox to provide IP Address Management
services. libnetwork is the library provided by Docker that allows third-party drivers for container
networking.


Prerequisite
------------
To use the driver, you need access to the Infoblox DDI product. For evaluation purposes, you can download a
virtual version of the product from the Infoblox Download Center (https://www.infoblox.com/infoblox-download-center)
Alternatively, if you are an existing Infoblox customer, you can download it from the support site.

Refer to CONFIG.md on how to configure vNIOS.

Build
-----
(refer to BUILD.md)

Installation
------------
By default, the ipam-driver assumes that the "Cloud Network Automation" licensed feature is activated. Should
this not be the case, refer to "Manual Configuration of Cloud Extensible Attributes" in CONFIG.md for additional
configuration required.

Run Executable
--------------
ipam-driver accepts a number of arguments which can be listed by specifying -h:

```
ubuntu$ ./ipam-driver --help
Usage of ./ipam-driver:
  -driver-name string
        Name of Infoblox IPAM driver (default "mddi")
  -global-network-container string
        Subnets will be allocated from this container when --subnet is not specified during network creation (default "172.18.0.0/16")
  -global-prefix-length uint
        The default CIDR prefix length when allocating a global subnet. (default 24)
  -global-view string
        Infoblox Network View for Global Address Space (default "default")
  -grid-host string
        IP of Infoblox Grid Host (default "192.168.124.200")
  -local-network-container string
        Subnets will be allocated from this container when --subnet is not specified during network creation (default "192.168.0.0/16")
  -local-prefix-length uint
        The default CIDR prefix length when allocating a local subnet. (default 24)
  -local-view string
        Infoblox Network View for Local Address Space (default "default")
  -plugin-dir string
        Docker plugin directory where driver socket is created (default "/run/docker/plugins")
  -ssl-verify string
        Specifies whether (true/false) to verify server certificate. If a file path is specified, it is assumed to be a certificate file and will be used to verify server certificate.
  -wapi-password string
        Infoblox WAPI Password
  -wapi-port string
        Infoblox WAPI Port. (default "443")
  -wapi-username string
        Infoblox WAPI Username
  -wapi-version string
        Infoblox WAPI Version. (default "2.0")
```

For example,

```
./ipam-driver --grid-host=192.168.124.200 --wapi-username=cloudadmin --wapi-password=cloudadmin --local-view=local_view --local-network-container="192.168.0.0/20,192.169.0.0/22" --local-prefix-length=25 --global-view=global_view --global-network-container="172.18.0.0/16" --global-prefix-length=24
```
The command need to be executed with root permission.

For convenience, a script called "run.sh" is provided which can be edited to specify the desired options.


Run Container
------------
Alternatively, the driver can also be run as a docker container.

A pre-built docker image can be pulled from Docker Hub using the following command:
```
docker pull infoblox/ipam-driver
```

After successfully pulling the image, you use the ```docker run``` command to run the driver. For exampe:
```
docker run -v /var/run:/var/run -v /run/docker:/run/docker infoblox/ipam-driver --grid-host=192.168.124.200 --wapi-username=cloudadmin --wapi-password=cloudadmin --local-view=local_view --local-network-container="192.168.0.0/20,192.169.0.0/22" --local-prefix-length=25 --global-view=global_view --global-network-container="172.18.0.0/16" --global-prefix-length=24
```

Note that the -v options are necessary to provide the container access to the specified directories on the
host file system.

For convenience, a script called "run-container.sh" is provided.

Usage
-----
To start using the driver, a docker network needs to be created specifying the driver using the --ipam-driver option:
```
sudo docker network create --ipam-driver=infoblox priv-net
```
This creates a docker network called "priv-net" which uses "infoblox" as the IPAM driver and the default "bridge"
driver as the network driver. A network will be automatically allocated from the list of network containers
specified during driver start up.

By default, the network will be created using the default prefix length specified during driver start up. You
can override this using the --ipam-opt option. For example:

```
sudo docker network create --ipam-driver=infoblox --ipam-opt="prefix-length=24" priv-net-2
```

After which, Docker containers can be started attaching to the "priv-net" network created above. For example,
the following command run the "ubuntu" image:

```
sudo docker run -i -t --net=priv-net --name=ubuntu1 ubuntu
```

When the container comes up, verify using the "ifconfig" command that IP has been successfully provisioned
from Infoblox.
