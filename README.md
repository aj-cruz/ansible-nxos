# ANSIBLE-NXOS
An Ansible playbook to deploy Cisco NXOS configurations to Nexus Switches.  
******************************************************************
**WARNING:** This playbook is designed to operate 100% declaratively.  
This means the variable definitions in Ansible are THE source of truth for the devices.  
If a variable is removed from the variable file, **IT WILL BE REMOVED FROM THE SWITCH CONFIGURATION** the next time the playbook is ran, even without the state set to "absent".  
If you will be using this playbook in your production environment, always perform a dry run first and test changes in a test/dev environment so you know exactly what will happen when it is ran in production.  

If you wish to change this behavior, you will need to dig through the modules and set any states from "overridden" to "merged" and remove any modules that delete configuration for resources that don't support the override feature.
****************************************************************** 

# REQUIREMENTS
- Python (Developed on 3.11.4)
- Python Packages:
    - Ansible (Developed on ansible-core 2.15.0)
    - ansible-pylibssh or paramiko (for SSH connections to devices)

You can install all the Python packages with: ```pip install -r requirements.txt```

# ANSIBLE.CFG
I made the following modifications to my ansible.cfg file to support the playbook:
- host_key_checking=False
- command_timeout=45  
(Every now and then one of my tasks that pipe output to json was taking a bit longer than the default 30 seconds)

# SUPPORTED NXOS CONFIGURATIONS
The playbook currently supports the following configurations:
- banners (motd & exec)
- clock (timezone & summer time)
- dns (domain lookup, domain name, name servers)
- features
- hostname
- logging (the basics)
- ntp
- vlan database (name & id)
- Non-Port Channel Trunk & Access Interfaces (description, admin state, allowed vlans, native vlan, access vlan, spanning tree port type)

More to come...

# DIRECTORY STRUCTURE
The directory structure follows Ansible best practices documented here: [Ansible Directory Layout Best Practices](https://docs.ansible.com/ansible/2.8/user_guide/playbooks_best_practices.html#directory-layout)
<pre>
├── group_vars/
|    ├── group1.yml                 # Here we define variables for groups in our inventory file
|    └── group2.yml
├── host_vars/
|    ├── hostname1.yml              # Here we define variables for individual systems in our inventory file
|    └── hostname2.yml
├── roles/
|    ├── common/                    # The common role is for general/global configurations such as hostname, NTP, logging, etc.
|    |    ├── tasks/
|    |    |    ├── main.yml         # Main task file that calls the individual subtasks
|    |    |    ├── subtask1.yml     # Subtask file that holds the Ansible logic for our task
|    |    |    └── subtask2.yml
|    |    └── templates/
|    |         ├── template1.j2     # Some subtask content is dynamically generated using JINJA templates defined here
|    |         └── template2.j2
|    └── vlans/                     # The vlan role is for configuring the vlan database (vlan ID & Name). Same directory structure as common
├── inventory.yml                   # The Ansible inventory file that defines device connectivity, group heirarchy, and devices
├── ansible.cfg                     # The Ansible configuration file (you can delete this if you will be using configuration from /etc/ansible/ansible.cfg)
└── main.yml                        # The Master Ansible Playbook. It scopes the hosts to run on and calls all the roles to perform the tasks of those roles
</pre>

# A NOTE ON THE VLAN ROLE
For scoping VLAN creation we could define the VLANs in the applicable group variable file, but the downsides of that approach are:
- VLAN database split across multiple locations (group vars)
- The same VLANs defined multiple times if it applies to multiple groups

I wanted a single location for my vlan database and I wanted to only have to define a VLAN one time.  
To that end, my approach is to put the vlan database in the ```all``` group, and use a custom field in my vlan database called ```apply_to_groups```  
The apply_to_groups field is a list that contains the device groups the VLAN should be configured on.  
In addition to inventory device groups, you can add an individual hostname should you require it.

# PREPARING TO RUN THE PLAYBOOK
To get the playbook ready to run in your environment:
1. Clone this repo to a Linux machine (The Control Node) with the Ansible/Python requirements
2. Configure management ports on your NXOS devices
3. Configure SSH credentials (username & password) on your NXOS devices and confirm you can SSH to all devices from the Control Node
4. Add environment variables to your Control Node to define the NXOS credentials in one of two ways:
    1. Manually define the credentials for your shell session by doing ```export NXOS_USER={{username}}``` and ```export NXOS_PASS={{password}}``` (supplying the actual username & password for {{username}} & {{password}} respectively)
    2. Create an environment file (such as environment.env). Place those exports in that file and source the file from your shell session by doing ```source environment.env```  
    **NOTE:** Option 2 is ok for a lab, but not recommended for production as your credentials are in plain-text in a flat file. For a better credential solution, the playbook can be modified to use a vault solution.
5. Update ```inventory.yml``` with settings relevant to your environment:
    - Device Groups
    - Hosts