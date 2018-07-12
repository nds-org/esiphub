# ESIPHub: Supporting Interactive Geoscience Workshops

ESIPhub is an incubator project of the [Earth Science Information Partners (ESIP) Lab](http://www.esipfed.org/esip-lab) initiative to develop a template for deploying a customized JupyterHub environment to support workshops at ESIP annual meetings.  Each year, scientists, engineers, educators, and practitioners convene to discuss their work, share ideas, and learn new tools and techniques. Workshops are often interactive and sometimes developed around applications such as Jupyter Notebooks. The goal of this project is to provide a framework for deploying JupyterHub using a standardized workshop configuration to provide predictability for instructors and reduce the time to setup.

Aspects of the project include:
* Configuration for deploying a Kubernetes cluster on OpenStack via the [Kubeadm Boostrapper with Terraform for OpenStack](https://github.com/nds-org/kubeadm-terraform) 
* Template configuration for deploying JupyterHub for ESIP via the [Zero-to-JupyterHub](https://zero-to-jupyterhub.readthedocs.io/) framework
* Authentication using pre-generated credentials
* Ability to deploy on both commercial cloud (AWS) and OpenStack environments (NCSA Nebula, SDSC Cloud, and XSEDE JetStream)
* Cost estimates for hosting ESIPhub for development and at scale for workshops
* Information sharing within the ESIP and NDS communities

As a result, the ESIP community will have a predictable method to deploy a custom JupyterHub environment for each workshop and understand the operational costs.

## Bootstrapping ESIPhub

The project team began by bootstrapping a [single-node Kubernetes environment on SDSC Cloud](docs/sdsc-bootstrap.md).  SDSC Cloud was selected because of resources available to the NDS group. The purpose of this environment was to minimize resource usage as the team became familiarized with the Kubernetes and Zero-to-JupyterHub paradigms. A single 2 vcpu and 8G virtual machine was sufficient for the first ~2 months of the project.  

The next step was to provision a [multi-node cluster on SDSC Cloud](docs/sdsc-zonca.md). For this, we followed a model defined by [Andrea Zonca for JupyterHub on Jetstream](https://zonca.github.io/2017/12/scalable-jupyterhub-kubernetes-jetstream.html) with some minor modifications. This allowed us to explore storage models, particulary the use of [Rook](https://rook.io/docs/rook/master/) to support persistent volume claims in OpenStack.

Building on this work and our experience [provisioning Kubernetes clusters via ansible](https://github.com/nds-org/ndslabs-deploy-tools), the NDS team took an initial pass at a [Terraform recipe](https://github.com/nds-org/kubeadm-terraform) for provisioning using Zonca's approach. We've now tested the Terraform process on NCSA Nebula, SDSC Cloud and TACC Jetstream OpenStack environments.


## Authentication
One of the original goals of the project was to support authentication using ESIP credentials.  However, after [some discsussion](https://github.com/nds-org/esiphub/issues/4) it was decided to move ahead with another model for this pilot project.  The ESIP annual meeting has ~400 attendees and any given workshop may have 60 or users. To simplify the process of running a workshop, the team decided to use predefined usernames and passwords that could be distributed to users at the meeting.

## Storage revisited
An ongoing challenge in the OpenStack environment is supporting storage. As usual, two separate needs arose: the ability to support per-container storage (i.e., ReadWriteOnce) versus shared storage (i.e., ReadWriteMany). The NDS team has spent a fair amount of energy looking at storage options for Kubernetes on OpenStack: [NFS](https://opensource.ncsa.illinois.edu/confluence/display/~lambert8/NFS+in+Kubernetes#NFSinKubernetes-BasicIn-ClusterServerExample), Gluster/Heketi, [Ceph/Rook](https://opensource.ncsa.illinois.edu/confluence/display/~lambert8/Rook), etc. The Rook model seems effective for single-user storage, but the shared filesystem model was less desirable (complicated and difficult to troubleshoot).  We opted to use Rook for the notebook PVCs and NFS for distributed shared storage across all environments. The [NSF Provisioner](https://opensource.ncsa.illinois.edu/confluence/display/~lambert8/NFS+in+Kubernetes) provides a viable alternative to rook.

## Other Configurations

* ESIPhub uses a [custom Docker image](dockerfiles/Dockerfile.scipy) which is autobuilt on [Dockerhub](https://hub.docker.com/r/ndslabs/esiphub-notebook/)
* Users will have 8GB of RAM and 10GB of storage in addition to the shared scratch space via NFS
* A custom `postStart` life-cycle hook copies a quickstart Readme and creates a symbolic link to the shared mount in each user's home directory.
* Kube-lego is used for TLS with DNS managed under the `ndslabs.org` domain.

## Moving from OpenStack to AWS
Next up...
