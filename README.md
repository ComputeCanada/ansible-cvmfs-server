# Prepare the system
It can be useful to do `echo "export PYTHONUNBUFFERED=1" >> ~/.bashrc` in your session to ensure that Ansible output is streamed to stdout smoothly.

## Set up ansible-pull
1. Enable EPEL repository

     EPEL is required to provide a recent version of Ansible and some JSON packages.
    * ```sudo dnf install epel-release```

1. Install Ansible

    Ansible v2.5 or higher is required. To install from EPEL:
    * ```sudo dnf install ansible```
    * It is useful to set ```force_color=True``` under ```[defaults]``` in ```/etc/ansible/ansible.cfg```

1. Install git
    * ```sudo dnf install git```

1.   Add gitlab server to system known hosts
     ```
     echo "git.computecanada.ca,199.241.165.30 ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILPNtdc21HhCqRQRZFQOHKyRHno/w58Y6OB047ZVS2TC" | sudo tee -a /etc/ssh/ssh_known_hosts
     ```
## Set up inventory
1.   Add the hostname of the new system to the Ansible inventory file ```hosts``` in git, with membership in the appropriate groups, and additional host or group vars as needed (which are documented [here](roles/cvmfs-server/defaults/main.yml)).


# Run ansible-pull
1.   Make sure you have logged in to the server using SSH agent forwarding (```ssh -A```) in order to access gitlab.
1.   `sudo` will be used by Ansible to execute the role - so do not use sudo when invoking `ansible-pull`. However you may need to e.g. `sudo -l` at this point to avoid a password prompt if applicable.
1.   It is good practice to use ```--check``` and ```--diff``` first to see which changes will be applied. 
     ```
     ansible-pull -U gitlab@git.computecanada.ca:cc-cvmfs/ansible-cvmfs-server.git --checkout=master --full -i hosts --check --diff
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

1.   A modification to the iptables file will be required, as described by the output of the iptables task if needed. 

# Testing and development
To test the Ansible role, create a branch in Git (named ```test``` in this example), and pull from there:

    ansible-pull -U gitlab@git.computecanada.ca:cc-cvmfs/ansible-cvmfs-server.git --checkout=test --full -i hosts
