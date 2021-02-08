To setup the server:

1. install debian 10
danach ist apt-get verfügbar,

2. ssh key erzeugen
ssh key generieren:
ssh-keygen

3. auf jedem Knoten ssh server installieren und konfigurieren:
sudo apt-get install openssh-server ii.
sudo service ssh status
vim /etc/ssh/sshd_config
PermitRootLogin yesssh
/etc/init.d/ssh restart

4. public key auf allen Knoten kopieren
ssh-copy-id root@10.211.55.4

danach auf remote einloggen:

ssh root@10.211.55.4
-> exit
ssh-copy-id root@10.211.55.4
-> exit

5. Ansible auf lokalem Recher installieren

sudo -H pip3 install ansible
check:
ansible --version

6. Ansible hosts
hosts

7. Ansible: create non root users
initial.yml

ansible-playbook -i hosts initial.yml  -v --extra-vars "ansible_sudo_pass=OSVHyuuvis2021!"

8. Ansible: install dependencies (docker) and the 3 kubernetes components kubeadm, kubelet and kubectl

Achtung: auf master: apt-get install acl

ansible-playbook -i hosts kube-dependencies.yml  -v --extra-vars "ansible_sudo_pass=OSVHyuuvis2021!"

9. Create the master node

ansible-playbook -i hosts master.yml  -v --extra-vars "ansible_sudo_pass=OSVHyuuvis2021!"

10. create workers nodes

ansible-playbook -i hosts ~/Optimal/kube-cluster/workers.yml


Problems:


docker info
shows: Cgroup Driver: cgroupsfs
but it should be: Cgroup Driver: systemd

In der Datei:
/usr/lib/systemd/system/docker.service
add --exec-opt....:
ExecStart=/usr/bin/dockerd --exec-opt native.cgroupdriver=systemd

and restart docker:

systemctl daemon-reload
systemctl restart docker

To delete configuration and/or data files of etcd and it's dependencies from Ubuntu Xenial then execute:
sudo apt-get autoremove --purge etcd

ssh jwewer@10.0.1.51

exort KUBECONFIG=/etc/kubernetes/admin.conf

sudo mkdir ~/.kube
sudo cp /etc/kubernetes/admin.conf ~/.kube/

cd ~/.kube

sudo mv admin.conf config
sudo service kubelet restart

kubectl apply -f "https://cloud.weave.works/k8s/v1.16/net.yaml?env.IPALLOC_RANGE=11.32.0.0/12"