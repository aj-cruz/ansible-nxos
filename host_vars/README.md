# ABOUT THIS FOLDER
Variables for individual hosts are defined in this folder.  
**They will override variables defined in the ```group_vars``` folder.**  

The files should be named with the format:
- {{inventory_hostname}}.yml  

Where {{inventory_hostname}} is the hostname defined in your Ansible inventory file.  
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
We could potentially have the following host variable files:
- CORE-1.yml
- LEAF-1.yml