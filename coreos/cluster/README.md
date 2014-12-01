# Azure CoreOS cluster deployment tool

[Deploying a CoreOS cluster on Azure](../../../master/coreos/cloud-init/README.md) takes a lot of manual steps using the Azure Cross platform CLI. This tool, built using the Azure Python SDK, automates these steps.

## Prerequisites

* Python > 2.7 and Pip
* Azure Python SDK > 0.9.0

You can [follow the installation instructions](http://azure.microsoft.com/en-us/documentation/articles/python-how-to-install/), or on a Mac:
```
brew install python
sudo pip install azure
```

Check prerequisites:
```
python -c "import azure; print(azure.__version__)"
0.9.0
```

* Azure subscription id and management certificate
This script uses the [Azure Service Management API from the Azure Python SDK](http://azure.microsoft.com/en-us/documentation/articles/cloud-services-python-how-to-use-service-management/). See the documentation for how to get your subscription id and certificate. The docs will also be useful  if you want to customize the script.
Read http://stackoverflow.com/questions/17705780/how-to-configure-ssh-login-with-key-pairs-for-a-linux-vm-using-azure-sdk-for-pyt for how to generate your ssh keys: pem, key, cer and sha1 thumbprint of the certificate.
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout mycert.key -out mycert.pem
openssl pkcs12 -export -in mycert.pem -inkey mycert.key -out mycert.pfx
openssl x509 -in mycert.pem -sha1 -noout -fingerprint | sed s/://g
SHA1 Fingerprint=44EF1BA225BE64154F7A55826CE56EA398C365B5
```

Warning: you will need 2 certificates in order to use this script: your azure management certificate, in pem format, and the ssh certificate in .cer format, as well as a sha1 thumbprint of this certificate.

## Quick Example

```
./azure-coreos-cluster pat-coreos-cloud-service \
--ssh-cert ~/.ssh/cert-with-ssh-public-key.cer \
--subscription 9b5910a1-8e79d5ea2841 \
--azure-cert ~/.azure/9b5910a1-8e79d5ea2841.pem \
--ssh-thumb 44EF1BA225BE64154F7A55826CE56EA398C365B5 \
--blob-container-url https://patcoreos.blob.core.windows.net/vhds/
```

Creates a 3 node cluster called pat-coreos-cloud-service.
use --num-nodes 5 option to bump it to create 5 instances instead of the default 3.

## Usage

```
Usage: azure-coreos-cluster [options] <cloud-service-name>

Options:

-h, --help                              output usage information
-V, --version                           output the version number
--subscription                          required, Azure subscription id
--azure-cert <pem-file>                 required, path to Azure cert pem file
--ssh-cert <Your pem file for ssh>      required, pem file with public key for ssh
--vm-size <size>                        optional, VM size [Small]
--vm-name-prefix <prefix>               optional, VM name prefix [coreos]
--availability-set <name>               optional, name of availability set for cluster [coreos-as]
--location <location>                   optional, [West US]
--ssh <ssh-port-start>                  optional, starts with 22001 and +1 for each machine in cluster
--virtual-network-name <vnet-name>      optional, name of an existing virtual network we place the cluster VMs in
--subnet-names <list>                   optional, subnet name
--custom-data <cloud-init-file.yml>     optional, your own cloud-init file, by default we use a here document in the python script, and generate a new discovery token
--coreos-image <image>                  optional, [2b171e93f07c4903bcad35bda10acf22__CoreOS-Beta-494.1.0]
--num-nodes <number>                    optional, number of nodes to create (or add), defaults to 3
--discovery-service-url <url>           optional, url for an existing cluster discovery service. Else we'll generate one.
--pip                                   optional, assigns public instance ip addresses to each VM
```

See official CoreOS Azure documentation [Running CoreOS on Azure](https://coreos.com/docs/running-coreos/cloud-providers/azure/) for CoreOS image names. Or install Azure CLI and run:
```
azure vm image list|grep CoreOS
```

[Instance-Level Public IP Addresses](http://msdn.microsoft.com/en-us/library/azure/dn690118.aspx) is right now in preview, and you are limited to 5 per subscription. The --pip option is useful for setting up a [CoreOS cluster for Deis](../../../master/coreos/deis/README.md). This option is not implemented yet.

For now, virtual-network and subnet options are not implemented yet.