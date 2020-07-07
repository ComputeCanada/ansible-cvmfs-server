# Compute Canada ansible-cvmfs-server role

This is a one-time example release of the Compute Canada ansible-cvmfs-server role, which is maintained elsewhere.
It may not immediately work out of the box, because some content has been removed.

# Prepare the system
## Set up ansible-pull
1. Enable EPEL repository

     EPEL is required to provide a recent version of Ansible, some JSON packages, and (on EL6) an Ansible dependency.
    * ```sudo yum install epel-release```

1. Install Ansible

    Ansible v2.5 or higher is required. To install from EPEL:
    * ```sudo yum install ansible```
    * It is useful to set ```force_color = 1``` in the defaults section of ```/etc/ansible/ansible.cfg```

1. Install git
    * ```sudo yum install git```

1.   Add gitlab server to system known hosts
     ```
     # For EL6:
     echo "git.computecanada.ca,199.241.165.30 ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC+wMyLTRvrSTwyaYYouRLghO5v2B27zm7HIkavggpkbtloRmTz2XWD90sMH+DaelKN6HgrpMeJI0zbp4bPtnNk3GXnSjx92BBZRmxFwZ3vEN77RQVuu1SdUTUp2Yuyz4h24Zh/YJDncRyQnTjzRYZ8op/X67WQJ4PYQkTU0BGmj69qw+3eg+hajkZQxZmaE7Bl5Vg7JGWIZbtdglLXXNhmQEsayUw4xZGpHlB31S4u/OHb1RIIpRc6sImwC2U+d7C8NhJHokbMRKFnaMFaWQyBN8wDmCjrtWlxOuSFpOJ0rZ6xO3PGWLfoEZNPPAGhWPq3GwTOSVQqaOB2vgU6JM/q5rwcgm6yRoN/JqaV5/dUjOOV9asmCQ7/NGfxYx2HMSxHlh7MhV0CMgZSA41T53d1fcUUj1dX2IsTrsF3qGF+g/5S8AR0owfkq5n57VWKZsBaNFh+mUv0rJDzpWAMPVKzQeNaVwZ9y9RpDHOjXKtW23AiTGVTbdEDPEr2asJ8eeGPevA6grw/WvToRUliUYUKILze60bAjkCwpLu5FKhxHiaiOPBS5W1a6DSP//EZgPVVfEaW6mBBJk2SX8/VIFbnvazdyYrvqkUBI1xCbONQBrnHAn01Ns4a8kgTKrs030ct2i/V06f/F8azTvJDXhOG1/Hp394uLB3RuMoCy+BF3w==" | sudo tee -a /etc/ssh/ssh_known_hosts
     # For EL7:
     echo "git.computecanada.ca,199.241.165.30 ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILPNtdc21HhCqRQRZFQOHKyRHno/w58Y6OB047ZVS2TC" | sudo tee -a /etc/ssh/ssh_known_hosts
     ```
## Set up inventory
1.   Add the hostname of the new system to the Ansible inventory file ```hosts``` in git, with membership in the appropriate groups, and additional host or group vars as needed (which are documented [here](roles/cvmfs-server/defaults/main.yml)).


# Run ansible-pull
1.   Make sure you have logged in to the server using SSH agent forwarding (```ssh -A```) in order to access gitlab.
1.   It is good practice to use ```--check``` first to see which changes will be applied. 
     ```
     ansible-pull -U gitlab@git.computecanada.ca:cc-cvmfs/ansible-cvmfs-server.git --checkout=master --full -i hosts --check
     ```
     Note: this may fail due to missing files etc., if the play has not yet completed in non-check mode to actually make changes, which is expected.
1.   Run without check mode to apply all changes.
     ```
     ansible-pull -U gitlab@git.computecanada.ca:cc-cvmfs/ansible-cvmfs-server.git --checkout=master --full -i hosts
     ```
1.   If you want to limit the scope of changes to a subset of tasks, use ```-t``` to run specific 
[tagged](https://docs.ansible.com/ansible/latest/user_guide/playbooks_tags.html) tasks. 
See [main.yml](roles/cvmfs-server/tasks/main.yml) for the list of tags that can be specified.

## Pre-requisites
Some parts of the stratum server deployment are not automated and must be done manually in order for the Ansible play to complete successfully.

1.   For a stratum 1 server, a license key for the MaxMind GeoIP2 DB is required. Acquire it from the MaxMind account or copy ```/etc/cvmfs/server.local``` from a pre-existing server to the new server.
1.   A modification to the iptables file will be required, as described by the output of the iptables task if needed. 

# Testing and development
To test the Ansible role, create a branch in Git (named ```test``` in this example), and pull from there:

    ansible-pull -U gitlab@git.computecanada.ca:cc-cvmfs/ansible-cvmfs-server.git --checkout=test --full -i hosts
