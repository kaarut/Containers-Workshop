# Prerequisites

To follow this interactive workshop, the following prerequisites should be met:

- OpenStack:
    - [Subscribe to IT's cloud service](https://clouddocs.web.cern.ch/tutorial_using_a_browser/subscribe_to_the_cloud_service.html) - we're going to use IT's OpenStack Cloud for this workshop.
    - Access to the [web interface of OpenStack](https://openstack.cern.ch/).
    - Personal Project (it should be automatically created once you request a personal project and subscribe to IT's cloud services).
    - An [SSH Key Pair](https://clouddocs.web.cern.ch/using_openstack/keypair_options.html#using-your-lxplus-ssh-key) on your Personal Project - this SSH Key Pair will be used to SSH to the Kubernetes cluster nodes.
    - Available quota needed for the Kubernetes cluster creation (4 instances in total; 1 master node and 3 worker nodes).

- `kubectl` CLI (see [installation instructions](https://kubernetes.io/docs/tasks/tools/)).

- (Optional) [OpenStack command-line clients](https://docs.openstack.org/newton/user-guide/common/cli-install-openstack-command-line-clients.html) to interact with the OpenStack API. We mainly need to install the following clients:
    - `python-openstackclient` - installs the OpenStack client
    - `python-magnumclient` - installs the client for Container Infrastructure Management Service

    !!! note
        OpenStack clients are also installed and can be used from lxplus. [Details](https://clouddocs.web.cern.ch/clients/lxplus.html).

- Internet connection to interact with the remote Kubernetes cluster.
