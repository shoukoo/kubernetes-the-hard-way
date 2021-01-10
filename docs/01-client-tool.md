# Install client tools
We will be using `master-1` to perform administrative task.

## Generate ssh keypair on master-1
We're going to generate ssh key pair and distribute it to all other VMs
```bash
ssh-keygen -f id_rsa
# Upload the key pair to /home/vagrant/.ssh on master-1 VM
vagrant upload  id_rsa /home/vagrant/ master-1
vagrant upload  id_rsa.pub /home/vagrant/ master-1
vagrant ssh --command "mv /home/vagrant/id_rsa.pub  /home/vagrant/.ssh/" master-1
vagrant ssh --command "mv /home/vagrant/id_rsa  /home/vagrant/.ssh/" master-1


# Upload public key to master-2
vagrant upload  id_rsa.pub /home/vagrant/ master-2
vagrant ssh --command "cat /home/vagrant/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys" master-2
# Upload public key to worker-1
vagrant upload  id_rsa.pub /home/vagrant/ worker-1
vagrant ssh --command "cat /home/vagrant/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys" worker-1
# Upload public key to worker-2
vagrant upload  id_rsa.pub /home/vagrant/ worker-2
vagrant ssh --command "cat /home/vagrant/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys" worker-2
# Upload public key to loadbalancer
vagrant upload  id_rsa.pub /home/vagrant/ loadbalancer
vagrant ssh --command "cat /home/vagrant/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys" loadbalancer
```
You can test the result by ssh onto master-1 and then from there try to ssh to other VMs.
```bash
vagrant ssh master-1
ssh master-2
ssh worker-1
ssh worker-2
ssh lb
```
## Install kubetl on master-1

```bash
wget https://storage.googleapis.com/kubernetes-release/release/v1.18.6/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

You can verify the version by typing the below command
```bash
kubectl version
```
