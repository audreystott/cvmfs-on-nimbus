# Cern-VM FS set-up

This is the documentation for setting up CernVM-FS on Nimbus. 

*If only required for accessing other Stratum 1s, skip steps A1, A2, B1, B2 and B3.*

A. Create 3 instances on Nimbus dashboard:

1. Stratum0 - n3.2c8r with 2 x 80 GB storage volume
2. Stratum1 - n3.2c8r with 2 x 80 GB storage volume
3. Caching-proxy - n3.1c4r

B. Follow the steps below for:

1. Installing cvmfs and cvmfs-server
2. Setting up Stratum 0
3. Setting up Stratum 1
4. Setting up the caching proxy
5. Setting up the client configuration

## Installing cvmfs and cvmfs-server

    wget https://ecsft.cern.ch/dist/cvmfs/cvmfs-release/cvmfs-release-latest_all.deb
    sudo dpkg -i cvmfs-release-latest_all.deb
    rm -f cvmfs-release-latest_all.deb
    sudo apt-get update
    sudo apt-get install cvmfs cvmfs-server

## Setting up Stratum 0

### Mounting storage volumes to cvmfs server locations

    sudo mkfs.ext4 /dev/vdc
    sudo mount /dev/vdc /var/spool/cvmfs
    
    sudo mkfs.ext4 /dev/vdd
    sudo mkdir /srv/cvmfs
    sudo mount /dev/vdd /srv/cvmfs

### Creating the repository

    sudo cvmfs_server mkfs containers.cvmfs.pawsey.org.au

    #Note: Root is Ubuntu, therefore owner of repo is Ubuntu

### Backing up the masterkey

### Enabling write access on repository   

    sudo cvmfs_server transaction

### Installing software on repository

### Publishing the repository

    sudo cvmfs_server publish

## Setting up Stratum 1

    sudo ./cvmfs-stratum-1-setup.sh \
         --stratum-0 146.118.70.122 \
        --servername stratum1-cvmfs.pawsey.org.au \
        --refresh 2 \
        pubkeys/containers.cvmfs.pawsey.org.au.pub

## Setting up the caching proxy

The caching proxy needs to have a security group with port 3218 opened to all IP addresses on the external network on Nimbus. This is so that the instances on these IP addresses can access data on CernVM-FS repositories.

To do so, create a new security group on the Nimbus dashboard following instructions for how to on this [page](https://support.pawsey.org.au/documentation/display/US/Allow+HTTPS+Access+To+Your+Instance#space-menu-link-content). Under the new security group, create a new rule that has the following settings:

- Rule: Custom TCP Rule
- Direction: Ingress
- Port: 3128
- Remote - CIDR: 0.0.0/0

On the cvmfs-proxy and cvmfs-proxy-2 instances, run the following:

        git clone https://github.com/audreystott/cvmfs-on-nimbus.git
        cd cvmfs-on-nimbus/
            sudo ./cvmfs-proxy-setup.sh \
                 --stratum-1 bcws.test.aarnet.edu.au \
                 --stratum-1 cvmfs1-mel0.gvl.org.au \
                 --stratum-1 cvmfs1-ufr0.galaxyproject.eu \
                 --stratum-1 cvmfs1-tacc0.galaxyproject.org \
                 --stratum-1 cvmfs1-iu0.galaxyproject.org \
                 --stratum-1 cvmfs1-psu0.galaxyproject.org \
                 146.118.64.0/21

        #Note that the proxy CIDR values correspond to all the external public IP addresses on Nimbus

On the cvmfs-proxy-3 instance, run:

        git clone https://github.com/cvmfs-on-nimbus.git
        cd cvmfs-on-nimbus/
        sudo ./cvmfs-proxy-setup.sh \
             --stratum-1 stratum1-cvmfs.pawsey.org.au \
             146.118.64.0/21

        #Note that the proxy CIDR values correspond to all the external public IP addresses on Nimbus

## Setting up the client configuration

For the client configuration, see https://github.com/PawseySC/Pawsey-CernVM-FS.git

## Notes

- For Stratum 0 only - to delete a repository, use:
    
      sudo cvmfs_server rmfs my.repo.name

- For more information on CernVM-FS, see here: https://cvmfs.readthedocs.io/en/stable/index.html
