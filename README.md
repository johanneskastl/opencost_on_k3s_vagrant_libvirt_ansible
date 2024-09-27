# opencost_on_k3s_vagrant_libvirt_ansible

Vagrant-libvirt setup that creates a VM with [k3s](https://k3s.io/), the minimal
lightweight Kubernetes distribution.

On top of k3s, this setup installs [OpenCost](https://www.opencost.io).

There is also an instance of Nginx running in the default namespace, to have
some workload.

Default OS is openSUSE Leap 15.6, but that can be changed in the Vagrantfile.
Please be aware, that this might break the Ansible provisioning.

## Note on the cost model

This setup uses a very simple cost model, where only CPU usage is being
"billed".

Feel free to change the cost model in the `playbook-opencost_installation.yml`
file in the `ansible` folder. The part you need to modify looks like this:

```
            customPricing:
              enabled: true
              costModel:
                description: only use CPU as indicator
                CPU: 1.00
                RAM: 0
                storage: 0
```

## Vagrant

1. You need `vagrant`, obviously. And `git`. And Ansible...
1. Fetch the box, per default this is `opensuse/Leap-15.6.x86_64`, using
   `vagrant box add opensuse/Leap-15.6.x86_64`.
1. Make sure the git submodules are fully working by issuing
   `git submodule init && git submodule update`
1. Run `vagrant up`
1. Open the URL that Ansible printed out at the end of the provisioning. It
   should look something like `http://opencost.192.168.2.13.sslip.io`.
1. Or you could run a command like the following:

```
kubectl --kubeconfig ansible/k3s-kubeconfig cost \
    --service-port 9003 \
    --service-name opencost \
    --kubecost-namespace opencost \
    --allocation-path /allocation/compute  \
    namespace \
    --historical \
    --window 5d \
    --show-cpu \
    --show-memory \
    --show-pv \
    --show-efficiency=false
```

1. The [documentation](https://www.opencost.io/docs/integrations/kubectl-cost)
   mentions another command to find the "total projected monthly costs based on
   the last two hours":

   ```
kubectl --kubeconfig ansible/k3s-kubeconfig cost \
    --service-port 9003 \
    --service-name opencost \
    --kubecost-namespace opencost \
    --allocation-path /allocation/compute  \
    namespace \
    --window 2h \
    --show-efficiency=true
   ```

1. Party!

## Cleaning up

The VMs can be torn down after playing around using `vagrant destroy`. This will
also remove the kubeconfig file `ansible/k3s-kubeconfig`.

## License

BSD-3-Clause

## Author Information

I am Johannes Kastl, reachable via git@johannes-kastl.de
