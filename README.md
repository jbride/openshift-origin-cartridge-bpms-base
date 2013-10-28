OpenShift JBPM ENGINE Cartridge
===============================


OVERVIEW
--------
  - Provides the JBPM process engine on OpenShift exposed via REST API.
  - This cartridge is essentially a fork of Bill Decoste's jbosseap cartridge.
  - Subsequently, most of administration guidelines documented in the jbosseap cartridge apply to this cartridge.


ACKNOWLEDGMENTS
---------------
  - many thanks to both the OpenShift and BPMS communities as well as Red Hat's Global Partner Enablement team for sponsoring this effort.


PURPOSE
-------
  - a JBPM engine cartridge is important because it allows for easy provisioning in a public and/or on-premise OpenShift cloud environment
    of the most important component of a BPM solution:  the BPM execution server.
  - in a production environment, this cartridge enables horizontal elasticity of a BPM execution server 'tier'
  - in a development environment, this cartridge enables easy provisioning of the same BPM execution server in a small, 'free-tier', cloud environment


FEATURES
--------
  - jbpm6 process engine exposed via the REST API provided by the droolsjbpm kie-services-remote project
  - process instance and human task persistence via a pre-configured 'jbpm' mysql database
  - droolsjbpm libraries deployed as EAP6 modules allowing for reduced runtime footprint
  - configurable knowledge session strategies:  PER_PROCESS_INSTANCE or SINGLETON
  - asynchronous (via embedded hornetq) BAM process event listener 
  - audit log persistence provided by a pre-configured 'jbpm_bam' postgresql-8.4 database
  - jbpm runtime has been slimmed down to fit comfortably in an Openshift small gear
  - will inter-operate with other cartridges from the Openshift bpmPaaS  suite:
    - business-central cartridge
    - dashboard cartridge
    - bam cartridge



GETTING STARTED
---------------
  - provision an OSE 1.2 on-premise environment or create an OpenShift Online account

  - provision BPMS6 Base
    - (if bpmsbase app already exists):  rhc app delete -a bpmsbase
    - rhc create-app bpmsbase "http://cartreflect-claytondev.rhcloud.com/reflect?github=jbride/openshift-origin-cartridge-bpms-base&commit=master"

  - execution server
    - rhc add-cartridge -a bpmsbase -c mysql-5.1
    - rhc add-cartridge -a bpmsbase "http://cartreflect-claytondev.rhcloud.com/reflect?github=jbride/openshift-addon-bpms-execution-server&commit=master" 

  - bounce the app
    - rhc app restart -a bpmsbase


  - test
     - there is currently a single KIE deployment at the following directory: $OPENSHIFT_DATA_DIR/runtime/deployments/general
     - that KIE deployment has two test process definitions:
           - BPMN2-MinimalProcess.bpmn2
           - BPMN2-UserTask.bpmn2
     - view existing processes:
        - curl -v -X GET http://my_openshift_app_ip/kie-jbpm-services/rest/additional/runtime/general/processes

     - start 'Minimal' process
        - curl -v -X POST http://my_openshift_app_ip/kie-jbpm-services/rest/runtime/general/process/Minimal/start

            
    
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

