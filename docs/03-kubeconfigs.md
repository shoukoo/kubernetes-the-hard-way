# Generating Kubernetes Configuration Files for Authentication
In this lab you will generate Kubernetes configuration files, also known as kubeconfigs, which enable Kubernetes clients to locate and authenticate to the Kubernetes API Servers.

## Client Authentication Configs
In this section you will generate kubeconfig files for the controller manager, kube-proxy, scheduler clients and the admin user.

Each kubeconfig requires a Kubernetes API Server to connect to. To support high availability the IP address assigned to the load balancer will be used. In our case it is 192.168.5.30

## The kube-proxy Kubernetes Configuration File
```
{
	export LOADBALANCER_ADDRESS=192.168.5.30
	kubectl config set-cluster kubernetes-the-hard-way \
		--certificate-authority=ca.pem \
		--embed-certs=true \
		--server=https://${LOADBALANCER_ADDRESS}:6443 \
		--kubeconfig=kube-proxy.kubeconfig

	kubectl config set-credentials system:kube-proxy \
		--client-certificate=kube-proxy.pem \
		--client-key=kube-proxy-key.pem \
		--embed-certs=true \
		--kubeconfig=kube-proxy.kubeconfig

	kubectl config set-context default \
		--cluster=kubernetes-the-hard-way \
		--user=system:kube-proxy \
		--kubeconfig=kube-proxy.kubeconfig

	kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
}
```

results:
```
kube-proxy.kubeconfig
```

## Kubelet Configuration File

```
LOADBALANCER_ADDRESS=192.168.5.30
for instance in worker-1 worker-2; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-credentials system:node:${instance} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${instance} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done
```

Results:
```
worker-1.kubeconfig
worker-2.kubeconfig
```

## The kube-controller-manager Kubernetes Configuration File
Kube controler manager are hosted in the master nodes, it can access the API server by hitting the 127.0.0.1:6443 address


Generate a kubeconfig file for the kube-controller-manager service:

```
{
	kubectl config set-cluster kubernetes-the-hard-way \
		--certificate-authority=ca.pem \
		--embed-certs=true \
		--server=https://127.0.0.1:6443 \
		--kubeconfig=kube-controller-manager.kubeconfig

	kubectl config set-credentials system:kube-controller-manager \
		--client-certificate=kube-controller-manager.pem \
		--client-key=kube-controller-manager-key.pem \
		--embed-certs=true \
		--kubeconfig=kube-controller-manager.kubeconfig

	kubectl config set-context default \
		--cluster=kubernetes-the-hard-way \
		--user=system:kube-controller-manager \
		--kubeconfig=kube-controller-manager.kubeconfig

	kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
}
```

Results:
```
kube-controller-manager.kubeconfig
```

## The kube-scheduler Kubernetes Configuration File
Generate a kubeconfig file for the kube-scheduler service:

```
{
	kubectl config set-cluster kubernetes-the-hard-way \
		--certificate-authority=ca.pem \
		--embed-certs=true \
		--server=https://127.0.0.1:6443 \
		--kubeconfig=kube-scheduler.kubeconfig

	kubectl config set-credentials system:kube-scheduler \
		--client-certificate=kube-scheduler.pem \
		--client-key=kube-scheduler-key.pem \
		--embed-certs=true \
		--kubeconfig=kube-scheduler.kubeconfig

	kubectl config set-context default \
		--cluster=kubernetes-the-hard-way \
		--user=system:kube-scheduler \
		--kubeconfig=kube-scheduler.kubeconfig

	kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
}
```

Results:
```
kube-scheduler.kubeconfig
```

## The admin Kubernetes Configuration File

```
{
	kubectl config set-cluster kubernetes-the-hard-way \
		--certificate-authority=ca.pem \
		--embed-certs=true \
		--server=https://127.0.0.1:6443 \
		--kubeconfig=admin.kubeconfig

	kubectl config set-credentials admin \
		--client-certificate=admin.pem \
		--client-key=admin-key.pem \
		--embed-certs=true \
		--kubeconfig=admin.kubeconfig

	kubectl config set-context default \
		--cluster=kubernetes-the-hard-way \
		--user=admin \
		--kubeconfig=admin.kubeconfig

	kubectl config use-context default --kubeconfig=admin.kubeconfig
}
```

Results:
```
admin.kubeconfig
```

## Distribute the Kubernetes Configuration Files

```
for instance in worker-1 worker-2; do
  scp  ${instance}.kubeconfig kube-proxy.kubeconfig ${instance}:~/
done

for instance in master-1 master-2; do
  scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig ${instance}:~/
done
```
