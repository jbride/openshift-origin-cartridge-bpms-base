bpmPaaS Cartridges using BPMS6
==============================

Overview
--------
  - Provisions scalable BPM application tier in Openshift
  - Allows for flexible deployment architecture to include an on-premise production environment
  - This cartridge is essentially a fork of Bill Decoste's jbosseap cartridge.
  - Subsequently, most of administration guidelines documented in the jbosseap cartridge apply to this cartridge.
  
  - rhc app delete -a bpmsbase
    - rhc create-app bpmsbase "http://cartreflect-claytondev.rhcloud.com/reflect?github=jbride/openshift-origin-cartridge-bpms-base&commit=master"

  - install 'add-on' cartridges
    
    - mysql cartridge :     rhc add-cartridge -a bpmsbase -c mysql-5.1

    - bpms high-availability server
        - rhc add-cartridge -a bpmsbase "http://cartreflect-claytondev.rhcloud.com/github/jbride/openshift-addon-bpms-ha"

    - bpms execution server
        - rhc add-cartridge -a bpmsbase "http://cartreflect-claytondev.rhcloud.com/reflect?github=jbride/openshift-addon-bpms-execution-server&commit=master" 

    - bpms business central
        - rhc add-cartridge -a bpmsbase "http://cartreflect-claytondev.rhcloud.com/reflect?github=jbride/openshift-addon-bpms-biz-central&commit=master" 



GENERAL ADMINISTRATION          
--------------------
  - bounce the app
    (executed from OS that has rhc client tools installed):   rhc app restart -a bpmsbase

  - tail the jbosseap log file
    - ssh into bpmsbase gear
    - tail -f bpmsbase/standalone/log/server.log


TEST
--------------------
    - there is currently various deployment units managed by the execution server as per:  exserver/filtered/kie.deployments.json
    - view list of REST API functions useful for trouble-shooting execution server:
        from root directory of gear:   ./exserver/bin/control printExServerRestCalls

  
    
    
TODO
----
  - Openshift related
    - package BPMS6 modules as RPMs ??
        - currently the size of the modules is 13 MB and is found in this cartridge at:   versions/6.0/modules/bpms.deployer.zip
    - allow for database dependency in cartridge
        - https://www.openshift.com/content/allow-for-database-dependency-in-cartridge

  - JBPM related
    - modules
        - deprecate existing modules in favor of official BPMS6 modules as described here:  https://bugzilla.redhat.com/show_bug.cgi?id=993126

    - DeploymentService strategy
        - currently using VFS DeploymentService backed by org.kie.commons.java.nio.fs.file.SimpleFileSystemProvider
        - should this cartridge continue to use VFS DeploymentService and switch to org.kie.commons.java.nio.fs.jgit.JGitFileSystemProvider ??
        - and / or
        - should this cartridge use a KModuleDeployment strategy where kjars are in a remote maven repo ??
            - could inter-operate with a different gear provisioned with guvnor-cartridge ?

    - audit logging
        - inject JMS AuditLogger from org.jbpm.process.audit.AuditLoggerFactory into kieSession
        - send process events to remote centralized JMS broker

