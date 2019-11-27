$script_setup = <<-SCRIPT
echo Updating your VM...
apt-get update
apt-get upgrade -y

echo Install Docker, kubectl, Firefox and some other tools...
apt-get install -y apt-transport-https gnupg-agent
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
add-apt-repository "deb https://apt.kubernetes.io/ kubernetes-xenial main"
add-apt-repository -y ppa:rmescandon/yq
apt-get update
apt-get install -y docker-ce docker-ce-cli containerd.io socat kubectl yq pkg-config bash-completion xauth xdg-utils firefox
usermod -a -G docker vagrant
su - vagrant -c "mkdir -p ~/.config && xdg-settings set default-web-browser firefox.desktop"
test $(grep kubectl ~vagrant/.bashrc | wc -l) -eq 0 && echo -e "\n# Enable kubectl shell autocompletion\nsource <(kubectl completion bash)" >> ~vagrant/.bashrc

echo Install Minikube, stern, kubectx...
rm -rf /usr/local/bin/minikube /usr/local/bin/stern ~vagrant/.kubectx
curl -fsSL -o /usr/local/bin/minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x /usr/local/bin/minikube
curl -fsSL -o /usr/local/bin/stern https://github.com/wercker/stern/releases/download/1.11.0/stern_linux_amd64 && chmod +x /usr/local/bin/stern
su - vagrant -c "git clone https://github.com/ahmetb/kubectx.git ~/.kubectx"
ln -sf ~vagrant/.kubectx/kubectx /usr/local/bin/kubectx
ln -sf ~vagrant/.kubectx/kubens /usr/local/bin/kubens
COMPDIR=$(pkg-config --variable=completionsdir bash-completion)
ln -sf ~vagrant/.kubectx/completion/kubens.bash $COMPDIR/kubens
ln -sf ~vagrant/.kubectx/completion/kubectx.bash $COMPDIR/kubectx
echo '{ "insecure-registries" : [ "10.96.0.0/12" ] }' > /etc/docker/daemon.json

echo Cleanup...
apt-get autoremove -y
SCRIPT

$script_startup = <<-SCRIPT
echo Starting Minikube...
sudo minikube start --vm-driver=none
sudo chown -R vagrant.vagrant ~/.kube ~/.minikube
for addon in dashboard registry; do
  test $(minikube addons list | grep ${addon}: | awk '{print $3}') == "disabled" && sudo minikube addons enable ${addon}
done
echo Done!
SCRIPT

Vagrant.configure(2) do |config|
  config.vm.define "ubukube-basic"
  config.vm.box = "bento/ubuntu-18.04"
  config.vm.hostname = "ubukube-basic"
  config.vm.network "private_network", ip: "192.168.56.101"
  config.vm.synced_folder '.', disabled: true
  config.vm.synced_folder 'work', '/home/vagrant/work', type: :virtualbox

  config.vm.provider :virtualbox do |vb|
    vb.name = "ubukube-basic"
    vb.gui = false
    vb.cpus = 4
    vb.memory = 8192
  end

  config.ssh.forward_agent = true
  config.ssh.forward_x11 = true

  config.trigger.before :halt do |trigger|
    trigger.info = "Stopping minikube..."
    trigger.run_remote = {inline: "sudo minikube stop", privileged: false}
  end

  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false

    config.trigger.after :up do |trigger|
      trigger.info = "Check vbguest version..."
      trigger.run = {inline: "vagrant vbguest --auto-reboot"}
    end
  end

  config.vm.provision "shell", inline: $script_setup
  config.vm.provision "shell", run: "always", privileged: false, inline: $script_startup
end
