# ABOUT THIS FOLDER
Variables for Ansible Inventory Groups are defined in this folder.  

The files should be named with the format:
- {{inventory_group}}.yml  

Where {{inventory_group}} is the name of a group defined in your Ansible inventory file. 
For example, if our inventory looks like this:
```yaml
all:
  children:
    core:
      hosts:
        CORE-1:
          ansible_host: <some_ip>
    leaf:
      hosts:
        LEAF-1:
          ansible_host: <some_ip>
```
We could potentially have the following group variable files:
- all.yml
- core.yml
- leaf.yml

**NOTE:** Variables defined in child groups will override those same variables defined in parent groups