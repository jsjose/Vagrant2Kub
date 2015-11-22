# Vagrant2Kub
Vagrant file for Kubernetes example http://www.projectatomic.io/blog/2015/09/new-centos-atomic-host-release-featuring-kubernetes-1-0/

This vagrantfile starts one master and 4 nodes using virtualbox provider. It uses "en0: Wi-Fi (AirPort)" as public network, then, if you need to use a different public, change it into the vagrantfile.

configFile directory contains configuration files of master and nodes. example directorio contains ngnix example. You can find a lot of examples in https://github.com/kubernetes/kubernetes.