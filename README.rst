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

Bootstrap salt-minion:

.. code-block:: bash

  export HTTPS_PROXY="http://proxy.your.corp:8080"; export HTTP_PROXY=$HTTPS_PROXY
  
  export MASTER_HOSTNAME=cfg01.infra.ci.local || export MASTER_IP=10.0.0.10
  export MINION_ID=$(hostname -f)             || export HOSTNAME=prx01 DOMAIN=infra.ci.local
  source <(curl -qL https://raw.githubusercontent.com/salt-formulas/salt-formulas-scripts/master/bootstrap.sh)
  install_salt_minion_pkg


Bootstrap salt-master:

.. code-block:: bash

  cd /srv/salt/scripts
  HOSTNAME=cfg01 DOMAIN=infra.ci.local ./bootstrap.sh

.. note:
  Creates /srv/salt/scripts/.salt-master-setup.sh.passed if succesfully passed the "setup script" 
  with the aim to avoid subsequent setup.


**salt-master-setup.sh** (DEPRECATED, use bootstrap.sh instead)

Script to install and configure salt *minion* but mostly *salt master* with *salt-formulas* common prerequisites in mind.
Configuration driven by environment variables, see source for more details...


**salt-master-init.sh** (DEPRECATED, use bootstrap.sh instead)

Script to bootstrap *salt master* and verify the model. To install salt master uses ``salt-master-setup.sh``.
Configuration driven by environment variables.


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
    export RECLASS_IGNORE_CLASS_NOTFOUND=False
    export EXTRA_FORMULAS="prometheus telegraph"

  
  
  
  


