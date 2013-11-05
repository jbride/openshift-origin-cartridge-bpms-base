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
    - allow for database dependency in cartridge
        - https://www.openshift.com/content/allow-for-database-dependency-in-cartridge
    - migrate to OSE 2.0 cartridge format

  - BPMS6 related

    - DeploymentService strategy
        - currently using PFP specific:  kie.deployments.json
        - migrate to whatever comes from: https://bugzilla.redhat.com/show_bug.cgi?id=1017327

    - split bpms 'core' and 'bam' JPA persistence units

    - update KnowledgeSession Entry Point API in biz-central to accept deployment ID ?
        - would avoid upfront look-up to identify deploymentId from processInstanceId (mapping contained in ProcessInstanceLog table)

    - implement alternative Task, and KnowledgeSession entry point implementations for biz central and dashboard web archives that use REST API from Execution Server
        - update B Tison's work to latest master branch and submit pull-request 

    - REST API enhancements
        - update B Tison's work to latest master branch and submit pull-request 

    - package BPMS6 modules as RPMs
        - https://bugzilla.redhat.com/show_bug.cgi?id=1005381

    - github home for these BPMS OSE cartridges ??

