# Vagrant2Kub
Vagrant file for Kubernetes example http://www.projectatomic.io/docs/gettingstarted/

This vagrantfile starts one master and 4 nodes using virtualbox provider. It uses "en0: Wi-Fi (AirPort)" as public network, then, if you need to use a different public interface, change it into the vagrantfile (but it works ok in windows).

configFile directory contains configuration files of master and nodes. example directorio contains ngnix example. You can find a lot of examples in https://github.com/kubernetes/kubernetes.

This version works over macOS and Windows (CygWin). I uses VirtualBox as provider (5.0.2r102096), vagrant 1.7.4.

Take in count in windows
- export VAGRANT_DETECTED_OS=cygwin

Currently working in scale size of VM in order to accept bigger examples from kubernetes project.