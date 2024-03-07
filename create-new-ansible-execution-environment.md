<pre>
  Execution environment components
The container images for the execution environments contain the necessary components to execute Ansible automation jobs. These include Python, Ansible (ansible-core), Ansible Runner, required Python libraries, and dependencies.

Image
Diagram of Ansible execution environment components: collections, libraries, and Ansible Core
(Source: Ansible's Inside Playbook blog)
When you install Ansible Automation Platform, the installer deploys the following container images whether you're in a connected or an unconnected installation:

The ee-29-rhel8 image contains Ansible 2.9 to use with older Ansible playbooks.
ee-minimal-rhel8 is the minimal container image with ansible-core and basic collections.
ee-supported-rhel8 is the container image with ansible-core and automation content collections supported by Red Hat.
Skip to bottom of list
Skip to the bottom of list
Automation advice
Ansible Automation Platform beginner's guide
A system administrator's guide to IT automation
Ansible Automation Platform trial subscription
Automate Red Hat Enterprise Linux with Ansible and Satellite
Use ansible-builder to create a custom execution environment
Ansible Automation Platform's default container images let you start doing automation without any additional configurations. If your automation requires additional libraries or plugins, you can build custom container images and store them inside any supported container registry, such as private automation hub.

You can follow the standard container image build process for building execution environment container images, but Ansible Automation Platform also includes a command-line utility called ansible-builder to build container images for custom execution environments. The ansible-builder tool can be installed from the upstream Python repository or the Red Hat RPM repository:

## Install ansible-builder utility
$ pip3 install ansible-builder
 
## Ansible Automation Platform repository subscription is required
$ sudo dnf install ansible-builder
The ansible-builder helps you build container images with the definition file execution-environment.yml. A typical execution-environment.yml contains the base container image (EE_BASE_IMAGE), ansible.cfg, and other dependency file details:

---
version: 1
 
build_arg_defaults:
  EE_BASE_IMAGE: 'automationhub22-1.lab.local/ee-minimal-rhel8:latest'
 
ansible_config: 'ansible.cfg'
 
dependencies:
  galaxy: requirements.yml
  python: requirements.txt
 
additional_build_steps:
  append:
    - RUN microdnf install which
In the preceding code, automationhub22-1.lab.local is the private automation hub, but you can use another container registry source, such as registry.redhat.io or Quay.io.

Once you've prepared the execution-environment.yml, execute the ansible-builder build command to create a build context that includes the Containerfile.

Now, create the container:

$  ansible-builder build --tag my_custom_ee
Running command:
  podman build -f context/Containerfile -t my_custom_ee context
Complete! The build context can be found at: /home/gmadappa/ansible-aap-demo/context
Refer to the official documentation to learn more about building container images using ansible-builder.

Skip to bottom of list
Skip to the bottom of list
Linux containers
A practical introduction to container terminology
Containers primer
Download now: Red Hat OpenShift trial
eBook: Podman in Action
Why choose Red Hat for containers
Use custom execution environments in unconnected environments
There are situations where the automation platform is inside an air-gapped environment or restricted network with limitations including:

No access to external Python repositories or no internal PyPI repository servers available to use
No access to required RPM repositories
In this case, you have two options to build and use custom execution environments with Ansible Automation Platform: building and transferring the container image or creating a custom environment.

1. Build and transfer a container image
You can create a container image from a connected machine (for example, a developer workstation) with all the dependencies inside and transfer it to the private automation hub (or another supported registry).

Step 1. Create and archive the container image from a connected machine:

## build the container image
$  ansible-builder build --tag my_custom_ee
 
## Save the container image as archive file
$  podman save --quiet -o my_custom_ee-1.0.tar localhost/my_custom_ee:1.0
Step 2. Copy the archived container image (for example, my_custom_ee-1.0.tar) to the target unconnected system using secure file transfer, a secure USB drive, or another method available to you.

Step 3. Load the container image from the TAR file to the system on the unconnected machine, and build the container image:

$ podman load -i my_custom_ee-1.0.tar
Step 4. Follow the tag and push process for private automation hub.

[ Take a no-cost technical overview course from Red Hat: Ansible essentials: Simplicity in automation. ]

2. Create a custom execution environment in an unconnected environment
You can create a container image inside the unconnected machine after transferring the dependencies.

Step 1. Transfer the dependencies to the target unconnected system using secure file transfer, a secure USB drive, or another method available to you.

For example, the following demonstration contains an archive file with the Python libraries required for my automation playbooks and collections:

 $  tar -tf python-packages.tar
python-packages/
python-packages/pan_os_python-1.7.3-py2.py3-none-any.whl
python-packages/ipaddress-1.0.23-py2.py3-none-any.whl
python-packages/requirements.txt
python-packages/pan_python-0.17.0-py2.py3-none-any.whl
Step 2. Prepare the Containerfile with instructions to build the container image for the execution environment:

## Containerfile for custom execution environment
ARG EE_BASE_IMAGE=registry.redhat.io/ansible-automation-platform-22/ee-minimal-rhel8:latest
ARG EE_BUILDER_IMAGE=registry.redhat.io/ansible-automation-platform-22/ansible-builder-rhel8

FROM $EE_BASE_IMAGE

ADD ansible.cfg ansible.cfg
ADD python-packages.tar python
RUN python3 -m pip install -r python/python-packages/requirements.txt --find-links=python/python-packages/ --no-index
Step 3. Build the container image using Podman:

$ podman build -f Containerfile -t localhost/network-ee:1.0
[...]

Looking in links: python/python-packages/
Processing ./python/python-packages/pan_os_python-1.7.3-py2.py3-none-any.whl
Processing ./python/python-packages/pan_python-0.17.0-py2.py3-none-any.whl
Installing collected packages: pan-python, pan-os-python
[...]

Successfully tagged localhost/network-ee:1.0
01e210e05a60dcf49c1b4a2b1bf1e58c49a487823b585233a15d1ecd66910bab
The TAR file is copied, extracted, and the content is installed inside the image.

Step 4. Follow the tag and push process for Private Automation Hub.

I'm making these assumptions about this process:

The ee-minimal-rhel8:latest and the ansible-builder-rhel8 container images are already on the unconnected machine (installed or configured as part of the Ansible Automation Platform deployment).
No multistage build is involved.
Push the container image to the private automation hub
You can store the container image in Ansible's private automation hub with other default execution environment images.

Step 1. Log in to the container registry:

$ podman login automationhub22-1.lab.local
Username: pahadmin
Password: 
Login Succeeded!
If you're using self-signed Secure Socket Layer (SSL) certificate on the private automation hub nodes, disable SSL validation by adding the --tls-verify=false option at the end.

Step 2. Tag the local container image with the private automation hub path:

$ podman tag localhost/network-ee:1.0 \
automationhub22-1.lab.local/network-ee:1.0
Step 3. Push the image to the private automation hub (registry):

$ podman push \
automationhub22-1.lab.local/network-ee:1.0
Step 4. Once the image is copied, verify the image on the private automation hub's graphical user interface:

Image
Screenshot of container repositories on a newly built execution environment image on Private Automation Hub
Gineesh Madapparambath, CC BY-SA 4.0
If you're not using the private automation hub, then you need to copy the container image to all automation controller nodes manually using the save and load method mentioned earlier.

Refer to the official documentation to learn how to use an execution environment in jobs on automation controller.

Skip to the bottom of list
Image
IT Automation ebook
Building containers
You can use any possible supported method to build container images for an execution environment and store it in a supported container registry. You can also use the upstream images (quay.io/ansible/ansible-runner:latest and quay.io/ansible/ansible-builder:latest) but with community-only support for building custom execution environment images.
  
</pre>
