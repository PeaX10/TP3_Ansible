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
![ping return](https://raw.githubusercontent.com/PeaX10/TP3_Ansible/master/img/ping_return.png)  
  
  
Now let's remove our Http server:  
![removing http](https://raw.githubusercontent.com/PeaX10/TP3_Ansible/master/img/remove_httpd.png)  
  

## Playbooks
Now here is our first playbook, yay !  
```yaml
- hosts : all
gather_facts : false
become : yes

tasks :
    - name : Test connection
      ping :
```  
Let's check the syntaxe then execute it:  
![check syntaxe playbook](https://raw.githubusercontent.com/PeaX10/TP3_Ansible/master/img/syntaxe_check.png)
![first playbook](https://raw.githubusercontent.com/PeaX10/TP3_Ansible/master/img/first_playbook.png)  
  
## Advanced Playbooks
```yaml
- hosts : all
gather_facts : false
become : yes
tasks :
    # Install Docker
    - name : Install yum-utils
    yum :
        name : yum-utils
        state : latest
    - name : Install device-mapper-persistent-data
    yum :
        name : device-mapper-persistent-data
        state : latest
    - name : Install lvm2
    yum :
        name : lvm2
        state : latest
    - name : Add Docker stable repository
    yum_repository :
        name : docker-ce
        description : Docker CE Stable - $basearch
        baseurl : https://download.docker.com/linux/centos/7/$basearch/stable
        state : present
        enabled : yes
        gpgcheck : yes
        gpgkey : https://download.docker.com/linux/centos/gpg
    - name : Install Docker
    yum :
        name : docker-ce
        state : present
    - name : Make sure Docker is running
        service : name=docker state=started
        tags : docker
```  
Now let's execute it: 
![advancedplaybook](https://raw.githubusercontent.com/PeaX10/TP3_Ansible/master/img/advanced_playbook.png)

*__What is $basesearch ?__*  
When a repository id is displayed, append these yum variables to the string if they are used in the baseurl/etc. Variables are appended in the order listed (and found).  
  
## Roles
```yaml
- name: Close All ports
  iptables:
    chain: INPUT
    policy: DROP
- name: Open HTTP port (80)
  firewalld:
    service: http
    permanent: yes
    state: enabled
- name: Open HTTP port (22)
  firewalld:
    service: ssh
    permanent: yes
    state: enabled
```  
For HTTPS traffic, you should open 443 TCP port.  
  
## Deploy our app
```yaml
- name: Run Database
  docker_container:
    name: database
    image: peax10/tp-cpe:db
- name: Run HTTPD
  docker_container:
    name: http
    image: peax10/tp-cpe:http
```  
