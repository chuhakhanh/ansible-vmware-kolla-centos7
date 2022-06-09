# ansible-vmware-kolla-centos7

# I - prepare centos-7-2009 minimal

yum install docker

## pip3
yum install -y python3
sudo yum install python3-pip
sudo pip3 install docker
pip3 install "kolla-ansible==9.0.*"

## pip
sudo yum install python-devel libffi-devel gcc openssl-devel libselinux-python
sudo yum install python-virtualenv
virtualenv /path/to/virtualenv
source /path/to/virtualenv/bin/activate
pip install -U pip
pip install -U setuptools
pip install 'ansible<2.10'
pip install "kolla-ansible==9.0.*"
mkdir /etc/kolla/
cp -r /path/to/virtualenv/share/kolla-ansible/etc_examples/kolla/* /etc/kolla
ssh-keygen
cd /path/to/virtualenv/share/kolla-ansible/ansible/inventory/
cat all-in-one 
ansible -i all-in-one all -m ping

## storage prepare 20G 
dd if=/dev/zero of=/var/loopbackfile.img bs=100M count=200

### Create or use losetup -d /dev/loop1 to remove
losetup -fP /var/loopbackfile.img
losetup -a
/dev/loop0: [64768]:76856 (/CentOS-7-x86_64-DVD-2009.iso)
/dev/loop1: [64768]:67379733 (/var/loopbackfile.img)
pvcreate /dev/loop1
vgcreate cinder-volumes /dev/loop1


# II - deploy openstack base centos-7-2009

source /venv_kolla-c2/bin/activate; cd /venv_kolla-c2/share/
## prepare openstack environment
ansible-playbook deploy_vms_kolla_cluster.yml 

for i in control-4 control-5 control-6;
do 
  sshpass -p alo1234 ssh-copy-id -f -i ~/.ssh/id_rsa.pub root@$i ; 
done


ansible-playbook -i all-in-one.cent7.control-4 prepare_all_node.yml
ansible-playbook -i all-in-one.cent7.control-4 prepare_storage_lvm.yml

cp kolla/globals.cent7.train.rating.yml kolla/globals.yml

kolla-ansible --configdir ./kolla -i ./all-in-one.cent7.control-4 bootstrap-servers
kolla-ansible --configdir ./kolla -i ./all-in-one.cent7.control-4 prechecks
kolla-ansible --configdir ./kolla -i ./all-in-one.cent7.control-4 deploy

### svc
cp kolla/globals.cent7.train.svc.yml kolla/globals.yml
kolla-ansible --configdir ./kolla -i ./all-in-one bootstrap-servers
