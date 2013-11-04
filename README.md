bpmPaaS Cartridges using BPMS6
==============================

Overview
------------------
* Provisions scalable BPM application tier in Openshift
* Allows for flexible deployment architecture to include an on-premise production environment as per this diagram

    - https://raw.github.com/jbride/openshift-origin-cartridge-bpms-base/master/doc/bpmPaaS_Overview/images/bpms6-deployment-architecture-openshift.png

* NOTE:  this cartridge differs from the "standalone" BPMS cartridge that packs all functionality into a single gear as per this diagram

    - https://raw.github.com/jbride/openshift-origin-cartridge-bpms-full/master/doc/images/bpmPaaS-standalone-deployment-architecture.png

*This cartridge is essentially a fork of Bill Decoste's jbosseap cartridge.  Subsequently, most of administration guidelines documented in the jbosseap cartridge apply to this cartridge.


INITIAL SETUP
-----------------
1.  Create an OSE 1.2 environment that can support a mix of small and medium gear sizes
2.  Install OSE 1.2 rhc tools

3.  Delete any previous bpms6 related OSE applications that you may have previously created

        - rhc app delete -a bpms

4.  Create the initial BPMS6 *base* OSE application

        - rhc create-app bpms "http://cartreflect-claytondev.rhcloud.com/reflect?github=jbride/openshift-origin-cartridge-bpms-base&commit=master"

5.  Install 'add-on' cartridges
    
    - mysql cartridge :     rhc add-cartridge -a bpms -c mysql-5.1

    - bpms high-availability server
        - rhc add-cartridge -a bpms "http://cartreflect-claytondev.rhcloud.com/github/jbride/openshift-addon-bpms-ha"

    - bpms execution server
        - rhc add-cartridge -a bpms "http://cartreflect-claytondev.rhcloud.com/reflect?github=jbride/openshift-addon-bpms-execution-server&commit=master" 

    - bpms business central
        - rhc add-cartridge -a bpms "http://cartreflect-claytondev.rhcloud.com/reflect?github=jbride/openshift-addon-bpms-biz-central&commit=master" 



GENERAL ADMINISTRATION          
--------------------
  - bounce the app
    (executed from OS that has rhc client tools installed):   rhc app restart -a bpmsbase

  - tail the jbosseap log file
    - ssh into bpmsbase gear
    - tail -f bpmsbase/standalone/log/server.log


TEST
--------------------
* There are currently various deployment units managed by the execution server as per:  exserver/filtered/kie.deployments.json
* View list of REST API functions useful for trouble-shooting execution server

    - from root directory of gear:   ./exserver/bin/control printExServerRestCalls

  
    
    
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

