# DevOps: TP3 - Ansible

## Inventories  
Default folder: ``/etc/ansible/hosts``  
```yaml
all :
    vars :
        ansible_user : centos
        ansible_ssh_private_key_file : /path/to/private/key
    children :
        prod :
            hosts : OVH server IP
```  
Then once we ping we get the pong:  
![ping return]()  

