# ansible-vmware-kolla-centos7

# I - prepare centos-7-2009 minimal
sudo yum install python-pip
sudo pip install docker
yum install -y python3


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

