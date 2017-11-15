==============
Deploy scripts
==============

This ``deploy/scripts`` serve as shared common place to link any deployment tools.


**bootstrap-salt.sh**
**bootstrap-salt.ps1**

Salt bootstrap scripts. Local copy of upstream `https://bootstrap.saltstack.com/`_.

**bootstrap.sh**

Script with function library to 
* install and configure *salt master* and *minions*
* bootstrap *salt master* with *salt-formulas* common prerequisites in mind
* validate reclass the model / pillar for all nodes

TL;DR:
======

Bootstrap salt-minion:

.. code-block:: bash

  export HTTPS_PROXY="http://proxy.your.corp:8080"; export HTTP_PROXY=$HTTPS_PROXY
  
  export MASTER_HOSTNAME=cfg01.infra.ci.local || export MASTER_IP=10.0.0.10
  export MINION_ID=$(hostname -f)             || export HOSTNAME=prx01 DOMAIN=infra.ci.local
  source <(curl -qL https://raw.githubusercontent.com/salt-formulas/salt-formulas-scripts/master/bootstrap.sh)
  install_salt_minion_pkg


Common procedure
================

Download the deploy scripts to the ``/srv/salt/scripts`` directory:

   .. code-block:: bash

      git clone https://github.com/salt-formulas/salt-formulas-scripts /srv/salt/scripts

Install reclass (optional):

.. note:: For bootstrap you may wish to use the forked version with some nice features (as ignore_class_notfound) from
          master or develop branch of https://github.com/salt-formulas/reclass.

.. code-block:: bash

  RECLASS_VERSION=dev
  cd /srv/salt/scripts
  source /srv/salt/scripts.bootstrap.sh
  install_reclass

.. note:: To ignore missing classes on bootstrap export the following variables
          ``export RECLASS_IGNORE_CLASS_NOTFOUND=True; export RECLASS_IGNORE_CLASS_REGEXP="service.*"``

If you are not using forked reclass (with ingnore_class_notfound option enabled) you have to set
environment variable FORMULAS_SALT_MASTER containing list of all formulas required on salt master.
For example you may require to pre-install the following:

.. code-block:: bash

  export FORMULAS_SALT_MASTER="linux salt reclass maas memcached openssh ntp  sphinx \
    grafana libvirt rsyslog glusterfs postfix xtrabackup freeipa prometheus telegraf \
    elasticsearch kibana rundeck devops-portal rsync docker keepalived aptly jenkins \
    gerrit artifactory influxdb horizon nginx collectd heka mysql"


Run the ``bootstrap.sh`` script from ``/srv/salt/scripts`` with the ``MASTER_HOSTNAME=$SALT_MASTER_FQDN`` parameter to
Bootstrap salt-master:

.. code-block:: bash

  cd /srv/salt/scripts
  CLUSTER_NAME=regionOne HOSTNAME=cfg01 DOMAIN=infra.ci.local ./bootstrap.sh

.. note:: Creates /srv/salt/scripts/.salt-master-setup.sh.passed if succesfully passed the "setup script"
          with the aim to avoid subsequent setup.


**formula-fetch.sh**

Script to install formulas with dependencies.


**salt-state-apply-trend.sh**

Simple script to invoking highstate on whole infrastructure with ``test=true``. Json output is aggregated with `jq`
(Failed/Success/Changes/Errors) and compared with previous run.


Bootstrap the Salt Master node
==============================
(expects salt-formulas reclass model repo)

.. code-block:: bash

  git clone https://github.com/salt-formulas/salt-formulas-scripts /srv/salt/scripts

  git clone <model-repository> /srv/salt/reclass
  cd /srv/salt/reclass
  git submodule update --init --recursive
  
  # OR (if system level is not add yet)
  git submodule add https://github.com/Mirantis/reclass-system-salt-model \
    /srv/salt/reclass/classes/system/

  cd /srv/salt/scripts
  HOSTNAME=cfg01 DOMAIN=infra.ci.local ./bootstrap.sh
  
  
Verify
------
Get the *verify.sh* script from https://github.com/salt-formulas/salt-formulas/tree/master/deploy/model

.. code-block:: bash

  cd /srv/salt/reclass
  HOSTNAME=cfg01 DOMAIN=infra.ci.local ./verify.sh          # or just ./verify.sh

  
  # individuall minions, if minions get generated under nodes/_generated
  ./verify.sh ctl01.k8s-cis-virtual.local
  
  
Additional bootstrap ENV variables
----------------------------------
(for full list of options see the *bootstrap.sh* source)
  
.. code-block:: bash

    # reclass
    export RECLASS_ADDRESS=<repo url>   ## if not already cloned in /srv/salt/reclass >
    export RECLASS_VERSION=dev

    # formula
    export FORMULAS_BRANCH=master
    export FORMULAS_SOURCE=git

    # system / host / salt master minion id
    export HOSTNAME=cfg01
    export DOMAIN=infra.ci.local
    # Following variables are calculated from the above if not provided
    #export MINION_ID
    #export MASTER_HOSTNAME
    #export MASTER_IP

    # salt
    export BOOTSTRAP_SALTSTACK_OPTS=" -dX stable 2016.3"
    export EXTRA_FORMULAS="prometeus"
    SALT_SOURCE=${SALT_SOURCE:-pkg}
    SALT_VERSION=${SALT_VERSION:-latest}
    
    # bootstrap
    export SALT_MASTER_BOOTSTRAP_MINIMIZED=False
    export CLUSTER_NAME=<%= cluster %>
    
    # workarounds
    export RECLASS_IGNORE_CLASS_NOTFOUND=True
    export RECLASS_IGNORE_CLASS_REGEXP="service.*"
    export EXTRA_FORMULAS="prometheus telegraph"

  
  
  
  


