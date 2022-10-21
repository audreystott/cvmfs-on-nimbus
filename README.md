# Cern-VM FS set-up

Documentation for setting up CernVM-FS on Nimbus. 

## From the Nimbus Dashboard

Create 5 instances on the Nimbus dashboard within the same public network:

1. Stratum0 - n3.2c8r with 2 x 80 GB storage volume
2. Stratum1 - n3.2c8r with 2 x 80 GB storage volume
3. Caching proxies - 3 x n3.2c8r

Create security groups as such:

1. Open port 80 on Stratum 0 for Stratum 1
2. Open port 80 and 8000 on Stratum 1 for proxies
3. Open port 3128 on proxies for the CIDR 146.118.64.0/21. Note that the CIDR values correspond to all the external public IP addresses on Nimbus.

## From the instances

### Mounting storage volumes to cvmfs paths for both the Stratum 0 and Stratum 1 

    sudo mkfs.ext4 /dev/vdc
    sudo mount /dev/vdc /var/spool/cvmfs
    
    sudo mkfs.ext4 /dev/vdd
    sudo mkdir /srv/cvmfs
    sudo mount /dev/vdd /srv/cvmfs

### Setting up Stratum 0

This example sets up a containers repository, with the name `containers.cvmfs.pawsey.org.au`, keeping in mind that this is not a DNS name. To create more repositories, run the following steps with the repository name required, in the same style and format as the containers one. It is best to be in keeping with `.cvmfs.pawsey.org.au` to ensure uniformity of CernVM-FS repos at Pawsey.

#### Running the provided script

This script installs all the required software and sets up the configuration scripts for CernVM-FS. The public key that is generated from this step will need to be copied to the Stratum 1 and proxy servers. For the purpose of this tutorial, the git repo has included the generated public key in the pubkeys folder.

    git clone https://github.com/audreystott/cvmfs-on-nimbus
    cd cvmfs-on-nimbus
    sudo cvmfs_server /cvmfs-stratum-0-setup.sh containers.pawsey.org.au
    
The generated public key `containers.cvmfs.pawsey.org.au.pub` can be found in the `/etc/cvmfs/keys` directory:

    > ls /etc/cvmfs/keys/
    containers.pawsey.org.au.pub

#### Enabling write access on repository   

    sudo cvmfs_server transaction

#### Installing software on repository

    cd /cvmfs/containers.pawsey.org.au

Proceed to install any software or copy files into this repository.

#### Publishing the repository

Run the publish command to make the changes permanent, this also ensures changes can't happen until the transaction command is run again as above.

    cd $HOME
    sudo cvmfs_server publish

### Setting up Stratum 1

Now that the Stratum 0 repository has been set up, the replica can be configured as below, keeping in mind that the public key was copied from the Stratum 0 server into the pubkeys folder here.

#### Running the provided script

    sudo ./cvmfs-stratum-1-setup.sh \
         --stratum-0 stratum0-cvmfs.pawsey.org.au \
        --servername stratum1-cvmfs.pawsey.org.au \
        --refresh 2 \
        pubkeys/containers.pawsey.org.au.pub

### Setting up the caching proxy

The proxies will also include repositories from AARNet and the Galaxy Project.

On the proxy instances, run the following:

        sudo ./cvmfs-proxy-setup.sh \
             --stratum-1 stratum1-cvmfs.pawsey.org.au \
             --stratum-1 bcws.test.aarnet.edu.au \
             --stratum-1 cvmfs1-mel0.gvl.org.au \
             --stratum-1 cvmfs1-ufr0.galaxyproject.eu \
             --stratum-1 cvmfs1-tacc0.galaxyproject.org \
             --stratum-1 cvmfs1-iu0.galaxyproject.org \
             --stratum-1 cvmfs1-psu0.galaxyproject.org \
             146.118.64.0/21

        #Note that the proxy CIDR values correspond to all the external public IP addresses on Nimbus

### Setting up the client configuration

For the client configuration, see https://github.com/PawseySC/Pawsey-CernVM-FS.git

## Notes

For Stratum 0 only - to delete a repository, use:
    
    sudo cvmfs_server rmfs my.repo.name

For more information on CernVM-FS, see here: https://cvmfs.readthedocs.io/en/stable/index.html

## Acknowledgements

The setup scripts have been kindly provided to us by QCIF. See here for more: [CernVM-FS setup](https://github.com/qcif/cvmfs-setup-example)
