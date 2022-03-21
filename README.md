# ansible-vmware-kolla-centos7

# II - deploy openstack base centos-7-2009

source /venv_kolla-c2/bin/activate
cd /venv_kolla-c2/share/
## prepare openstack environment
ansible-playbook deploy_vmware_rating_cluster.yml 

for i in control-4 control-5 control-6;
do 
  sshpass -p alo1234 ssh-copy-id -f -i ~/.ssh/id_rsa.pub root@$i ; 
done

kolla-ansible --configdir ./ -i ./all-in-one-control-3 bootstrap-servers
kolla-ansible -configdir ./ -i ./all-in-one-control-3 prechecks
kolla-ansible -configdir ./ -i ./all-in-one-control-3 deploy