rhc app delete -a bpmsbase
    - rhc create-app bpmsbase "http://cartreflect-claytondev.rhcloud.com/reflect?github=jbride/openshift-origin-cartridge-bpms-base&commit=master"

  - install 'add-on' cartridges
    
    - mysql cartridge :     rhc add-cartridge -a bpmsbase -c mysql-5.1

    - bpms high-availability server
        - rhc add-cartridge -a bpmsbase "http://cartreflect-claytondev.rhcloud.com/github/jbride/openshift-addon-bpms-ha"

    - bpms execution server
        - rhc add-cartridge -a bpmsbase "http://cartreflect-claytondev.rhcloud.com/reflect?github=jbride/openshift-addon-bpms-execution-server&commit=master" 




  - test
     - there is currently a single KIE deployment at the following directory: $OPENSHIFT_DATA_DIR/runtime/deployments/general
     - that KIE deployment has two test process definitions:
           - BPMN2-MinimalProcess.bpmn2
           - BPMN2-UserTask.bpmn2
     - view existing processes:
        - curl -v -X GET http://my_openshift_app_ip/kie-jbpm-services/rest/additional/runtime/general/processes

     - start 'Minimal' process
        - curl -v -X POST http://my_openshift_app_ip/kie-jbpm-services/rest/runtime/general/process/Minimal/start

  
OTHER ADMINISTRATION          
--------------------
  - bounce the app
    - rhc app restart -a bpmsbase
    
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

