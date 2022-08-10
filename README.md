# I - Prepare

## for os
centos-7-2009 minimal
## storage prepare for cinder-volumes 20G 

  dd if=/dev/zero of=/var/loopbackfile.img bs=100M count=200
  losetup /dev/loop2 /var/loopbackfile.img
  echo "losetup /dev/loop2 /var/loopbackfile.img; exit 0;" > /etc/init.d/cinder-setup-backing-file
  chmod 755 /etc/init.d/cinder-setup-backing-file
  ln -s /etc/init.d/cinder-setup-backing-file /etc/rc2.d/S10cinder-setup-backing-file
  pvcreate /dev/loop2
  vgcreate cinder-volumes /dev/loop2

# II - Setup
## A - with python2(legacy) 
yum install docker
sudo yum install python-devel libffi-devel gcc openssl-devel libselinux-python
sudo yum install python-virtualenv
yum install python-pip

### virtual environment
virtualenv /path/to/virtualenv
source /path/to/virtualenv/bin/activate
pip install -U pip
pip install -U setuptools
pip install 'ansible<2.10'
pip install "kolla-ansible==9.0.*"

### kolla preparation
mkdir /etc/kolla/
cp -r /path/to/virtualenv/share/kolla-ansible/etc_examples/kolla/* /etc/kolla
ssh-keygen
cd /path/to/virtualenv/share/kolla-ansible/ansible/inventory/
cat all-in-one 
ansible -i all-in-one all -m ping
source /venv_kolla-c2/bin/activate; cd /venv_kolla-c2/share/
### prepare openstack environment
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

## B - With python3 (after 6/2022 in test SVC)

### Note
After 6/2022, python2 help problem with docker SDK. 
Kolla-ansible need to deploy with python3. 
### pip3
  yum install -y python3
  sudo yum install python3-pip

### virtual env
virtualenv --python=python3 /path/to/virtualenv_python3
source /path/to/virtualenv_python3/bin/activate 
sudo pip3 install docker
pip3 install "kolla-ansible==9.3.2"
yum install libselinux-python3
update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.6 1
pip install python-openstackclient

### openstack deploy

cp kolla/globals.cent7.train.svc.yml kolla/globals.yml
kolla-ansible --configdir ./kolla -i ./all-in-one bootstrap-servers -e 'ansible_python_interpreter=/usr/bin/python3'
kolla-ansible --configdir ./kolla -i ./all-in-one prechecks -e 'ansible_python_interpreter=/usr/bin/python3'
kolla-ansible --configdir ./kolla -i ./all-in-one deploy -e 'ansible_python_interpreter=/usr/bin/python3'
