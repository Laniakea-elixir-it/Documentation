Cloud Images Creation
=====================

Laniakea exploits Cloud images for Express builds, i.e. the deployment of application which are pre-installed on a cloid images, and just reconfigured and started at deployment time.

These images needs to be rebuilt every time a new software release or update is delivered, and, of course, in case of critical security issues. Therefore, in the long run, their manual creation was not a sustainable strategy.

We use `Hashicorp Packer <https://developer.hashicorp.com/packer>`_ to create images, whith OpenStack plugin, to instantiate the VM on openstack cloud and save it automatically on it, and Ansible for software installation and configuration.

The `Cloud image module <https://github.com/Laniakea-elixir-it/laniakea-ci-infrastructure/tree/master/cloud-images>`_ encompasses:

#. a `JenkinsFile <https://raw.githubusercontent.com/Laniakea-elixir-it/laniakea-ci-infrastructure/master/cloud-images/JenkinsFile>`_ for jenkins integration;

#. a `YAML file <https://raw.githubusercontent.com/Laniakea-elixir-it/laniakea-ci-infrastructure/master/cloud-images/images_list.yaml>`_ listing the image to create and Ansible inputs.

#. a `python <https://raw.githubusercontent.com/Laniakea-elixir-it/laniakea-ci-infrastructure/master/cloud-images/build_images.py>`_ script for creating the image with Packer, which build the Packer json file and run it;

#. the Ansible playbooks and corresponding variables for application installation, i.e. Galaxy, RStudio, Jupyther and more.




Finally, the created images are shared among all OpenStack tenant with stored on CMDB
