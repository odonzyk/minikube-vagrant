# Minikube on Ubuntu

Using [Vagrant](https://www.vagrantup.com/) and [VirtualBox](https://www.virtualbox.org/) this `Vagrantfile` can be used to start a virtual machine that will automatically install [Minikube](https://kubernetes.io/docs/tutorials/hello-minikube/), [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) and some other useful tools (kubectx + kubens: Power tools for kubectl) during provisioning. During each start of the virtual machine with the `Vagrantfile` the Kubernetes cluster on Minikube is automatically started and the cluster is stopped when the VM is halted.

In order to be able to run `minikube dashboard` to show the Kubernetes dashboard of the cluster in a browser on the virtual machine the `Vagrantfile` also provisions the [Firefox](https://www.mozilla.org/firefox) browser and the required X Window packages. To actually show the browser a X server on the host system is required. This can be easily accomplished by using [MobaXterm](https://mobaxterm.mobatek.net/) as the SSH client to login to the virtual machine.

To use this VM perform the following steps (tested on Windows 10 Pro):

1. When necessary install [Vagrant](https://www.vagrantup.com/) and [VirtualBox](https://www.virtualbox.org/).
2. Clone this repository and switch to the right branch.
3. In the root folder of the clone repository run `vagrant up`.
4. When the VM is provisioned it's advised to restart it by running `vagrant halt` and then `vagrant up` again.
5. Login to the VM using `vagrant ssh` or use a SSH client such as [MobaXterm](https://mobaxterm.mobatek.net/).
6. When ready logout from the VM and stop it using `vagrant halt`.

When using a SSH client one can login to the VM with user `vagrant` and password `vagrant`. Instead of using a password one can also use the `private_key` created in subfolder `.vagrant/machines/default/virtualbox`.
