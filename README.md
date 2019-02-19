# openshift-ansible_step-by-step


master 4 VCPU  16GB Ram 80 GB HDD
node1 2 VCPU  8GB Ram  80 GB HDD 
node2 2 VCPU  8GB Ram  80 GB HDD

############################################################################
### PS: Configurar IP / HOSTNAME / SEARCH DOMAIN na instalação do CentOS ###
############################################################################

### Prepare todos os Hosts - Master e Node:

hostnamectl set-hostname master.okd.os
hostnamectl set-hostname node1.okd.os
hostnamectl set-hostname node2.okd.os

search okd.os
nameserver 8.8.8.8

hostname master.okd.os
hostname node1.okd.os
hostname node2.okd.os

HOSTS="$(head -n3 /etc/hosts)"
echo -e "$HOSTS" > /etc/hosts
echo -e '192.168.10.2 master.okd.os\n192.168.10.3 node1.okd.os\n192.168.10.4 node2.okd.os' > /etc/hosts

yum update -y

yum install -y wget git zile nano net-tools docker-1.13.1 bind-utils iptables-services bridge-utils bash-completion 
yum install -y kexec-tools sos psacct openssl-devel httpd-tools NetworkManager python-cryptography python2-pip python-devel python-passlib
yum install -y java-1.8.0-openjdk-headless "@Development Tools"
yum install -y curl vim device-mapper-persistent-data lvm2 epel-release wget git net-tools bind-utils yum-utils iptables-services bridge-utils bash-completion kexec-tools sos psacct docker-1.13.1-75.git8633870.el7.centos

yum update -y

cd /etc/yum.repos.d && curl -O https://storage.googleapis.com/origin-ci-test/releases/openshift/origin/master/origin.repo

systemctl enable NetworkManager
systemctl start  NetworkManager
systemctl status NetworkManager

############ Aplicar somente no Master ############

ssh-keygen -t rsa
ssh-copy-id root@master.okd.os
ssh-copy-id root@node1.okd.os
ssh-copy-id root@node2.okd.os


mkdir -p /etc/origin/master
touch /etc/origin/master/htpasswd
htpasswd -b /etc/origin/master/htpasswd admin password

cd ~
git clone https://github.com/openshift/openshift-ansible
cd openshift-ansible
git checkout release-3.11

curl -o ansible.rpm https://releases.ansible.com/ansible/rpm/release/epel-7-x86_64/ansible-2.6.5-1.el7.ans.noarch.rpm
yum -y  install ansible.rpm

##################################################
### Iserir configuração no /etc/ansible/hosts: ###
##################################################

[OSEv3:children]
masters
nodes
etcd

[OSEv3:vars]
ansible_ssh_user=root
openshift_deployment_type=origin
openshift_enable_olm=false
openshift_release=3.11
openshift_cluster_monitoring_operator_install=false
openshift_metrics_install_metrics=false
openshift_logging_install_logging=false
ansible_service_broker_install=false
template_service_broker_install=false
openshift_disable_check=disk_availability,memory_availability,docker_storage
openshift_disable_check=package_version

[masters]
master.okd.os

[etcd]
master.okd.os

[nodes]
master.okd.os openshift_node_group_name='node-config-master-infra' openshift_ip='192.168.10.2' openshift_public_ip='192.168.10.2' openshift_public_hostname='master.okd.os'
node1.okd.os openshift_node_group_name='node-config-compute' openshift_ip='192.168.10.3' openshift_public_ip='192.168.10.3' openshift_public_hostname='node1.okd.os'
node2.okd.os openshift_node_group_name='node-config-compute' openshift_ip='192.168.10.4' openshift_public_ip='192.168.10.4' openshift_public_hostname='node2.okd.os'

##################################################

--------------------------------------------------------------------
Master imagens (Aplicar somente no master):

docker pull docker.io/cockpit/kubernetes
docker pull docker.io/openshift/origin-deployer:v3.11
docker pull docker.io/openshift/origin-docker-registry:v3.11
docker pull docker.io/openshift/origin-haproxy-router:v3.11
docker pull docker.io/openshift/origin-pod:v3.11

docker pull docker.io/openshift/origin-control-plane:v3.11
docker pull quay.io/coreos/etcd:v3.2.22


Nodes imagens:

docker pull docker.io/cockpit/kubernetes
docker pull docker.io/openshift/origin-deployer:v3.11
docker pull docker.io/openshift/origin-docker-registry:v3.11
docker pull docker.io/openshift/origin-haproxy-router:v3.11
docker pull docker.io/openshift/origin-pod:v3.11

--------------------------------------------------------------------

# Executar no local em que foi feito o clone do repositório 'openshift-ansible' no meu caso em /root/

ansible-playbook openshift-ansible/playbooks/prerequisites.yml
ansible-playbook openshift-ansible/playbooks/deploy_cluster.yml


# Em caso de falha remover com playbook: 

ansible-playbook openshift-ansible/playbooks/adhoc/uninstall.yml



--------------------------------------------------------------------
