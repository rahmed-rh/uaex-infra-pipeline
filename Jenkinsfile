openshift.withCluster() {
 openshift.withCredentials('cluster-admin-credential-id') {
  def PROJECT_NAME = "ocplearning-project"
  def amqTemplate
  def amqModels
  def jdgModels
  def decisionserverModels
  def cm
  openshift.withProject(PROJECT_NAME) {
   def FIS_1_GIT_URL = "https://github.com/rahmed-rh/uaex-fis-1"
   def FIS_1_APP_NAME = "uaex-fis-1"
   def FIS_2_GIT_URL = "https://github.com/rahmed-rh/uaex-fis-2"
   def FIS_2_APP_NAME = "uaex-fis-2"
   def FIS_IMAGE_STREAM = "https://raw.githubusercontent.com/jboss-fuse/application-templates/GA/fis-image-streams.json"
   def AMQ_IMAGE_STREAM = "https://raw.githubusercontent.com/jboss-openshift/application-templates/master/amq/amq63-image-stream.json"
   def AMQ_TEMPLATE = "https://raw.githubusercontent.com/jboss-openshift/application-templates/master/amq/amq63-persistent.json"
   def AMQ_APP_NAME = "broker-uaex"
   def JDG_IMAGE_STREAM = "https://raw.githubusercontent.com/jboss-openshift/application-templates/master/datagrid/datagrid71-image-stream.json"
   def JDG_TEMPLATE = "https://raw.githubusercontent.com/jboss-openshift/application-templates/master/datagrid/datagrid71-basic.json"
   def JDG_APP_NAME = "uaex-jdg-data"
   def DECISIONSERVER_IMAGE_STREAM = "https://raw.githubusercontent.com/jboss-openshift/application-templates/master/decisionserver/decisionserver64-image-stream.json"
   def DECISIONSERVER_TEMPLATE = "https://raw.githubusercontent.com/jboss-openshift/application-templates/master/decisionserver/decisionserver64-basic-s2i.json"


   echo "Hello from project ${openshift.project()} in cluster ${openshift.cluster()}"

   // Mark the code checkout 'stage'....
   stage('Read CM') {

    def cmSelector = openshift.selector("configmap", "pipeline-app-config")
    def cmExists = cmSelector.exists()

    if (cmExists) {
     cm = cmSelector.delete()
    }
    def cmappconfig = [
     [
      "kind": "ConfigMap",
      "apiVersion": "v1",
      "metadata": [
       "name": "pipeline-app-config"
      ],
      "data": [
       "fis-1-app-git-url": "${FIS_1_GIT_URL}",
       "fis-1-app-name": "${FIS_1_APP_NAME}",
       "fis-2-app-git-url": "${FIS_2_GIT_URL}",
       "fis-2-app-name": "${FIS_2_APP_NAME}",
       "fis-image-stream": "${FIS_IMAGE_STREAM}",
       "amq-image-stream": "${AMQ_IMAGE_STREAM}",
       "amq-template": "${AMQ_TEMPLATE}",
       "amq-app-name": "${AMQ_APP_NAME}",
       "jdg-template": "${JDG_TEMPLATE}",
       "jdg-image-stream": "${JDG_IMAGE_STREAM}",
       "jdg-app-name": "${JDG_APP_NAME}",
       "decisionserver-template": "${DECISIONSERVER_TEMPLATE}",
       "decisionserver-image-stream": "${DECISIONSERVER_IMAGE_STREAM}"
      ]
     ]
    ]
    cm = openshift.create(cmappconfig).object()


    echo "The CM is ${cm}"
    echo "The fis-1-app-git-url is ${cm.data['fis-1-app-git-url']}"
    echo "The fis-2-app-git-url is ${cm.data['fis-2-app-git-url']}"

   }

  }

  openshift.withProject("openshift") {

   // Should i really check if template or is exists or no, or just go with the latest as the is maybe update !!!
   //create FIS builder imagestream
   //def fisISSelector = openshift.selector("imagestream", "fis-java-openshift")
   //def fisISExists = cmSelector.exists()
   //if (!fisISExists) {
   //openshift.replace("--force", "-f ", cm.data['fis-image-stream'])
   //}

   //create AMQ builder imagestream & template
   //def amqTemplateSelector = openshift.selector("template", "amq63-persistent")
   //def amqTemplateExists = amqTemplateSelector.exists()
   //if (!amqTemplateExists) {
   //amqTemplate = openshift.replace("--force", "-f ", cm.data['amq-template'])
   //}

   //def amqISSelector = openshift.selector("imagestream", "amq63-image-stream")
   //def amqISExists = amqISSelector.exists()
   //if (!amqISExists) {
   //openshift.replace("--force", "-f ", cm.data['amq-image-stream'])
   //}

   echo "Creating/updating FIS ImageStream"
   openshift.replace("--force", "-f ", cm.data['fis-image-stream'])

   echo "Creating/updating AMQ ImageStream & Template"
   openshift.replace("--force", "-f ", cm.data['amq-image-stream'])
   amqTemplate = openshift.replace("--force", "-f ", cm.data['amq-template'])
 
   echo "Creating/updating JDG ImageStream & Template"
   openshift.replace("--force", "-f ", cm.data['jdg-image-stream'])
   jdgTemplate = openshift.replace("--force", "-f ", cm.data['jdg-template'])

   echo "Creating/updating DecisionServer ImageStream & Template"
   openshift.replace("--force", "-f ", cm.data['decisionserver-image-stream'])
   openshift.replace("--force", "-f ", cm.data['decisionserver-template'])

   echo "Processing AMQ Template"
   amqModels = openshift.process("amq63-persistent", "-p APPLICATION_NAME=${cm.data['amq-app-name']} ", "-p AMQ_STORAGE_USAGE_LIMIT=5gb", "-p MQ_USERNAME=admin", "-p MQ_PASSWORD=passw0rd", "-p MQ_QUEUES=TESTQUEUE")
   echo "Processing JDG Template"
   jdgModels = openshift.process("datagrid71-basic", "-p APPLICATION_NAME=${cm.data['jdg-app-name']}", "-p INFINISPAN_CONNECTORS=hotrod,rest", "-p HOTROD_AUTHENTICATION=true", "-p CACHE_NAMES=default,payees", "-p USERNAME=admin", "-p PASSWORD=redhat1!")
  }

  openshift.withProject(PROJECT_NAME) {
   stage('Preparing AMQ Dev Env') {
    //create amq servie account
    def amqSASelector = openshift.selector("serviceaccount", "amq-service-account")
    def amqSAExists = amqSASelector.exists()
    if (!amqSAExists) {
     openshift.create('serviceaccount', 'amq-service-account')
     openshift.policy("add-role-to-user", "view", "system:serviceaccount:$PROJECT_NAME:amq-service-account", "-n", PROJECT_NAME)
    }
    echo "Deleting OLD AMQ Env if exist"
    openshift.selector( 'all', [ application: cm.data['amq-app-name'] ] ).delete()
    openshift.selector( 'pvc', [ application: cm.data['amq-app-name'] ] ).delete()
    
    echo "Create New AMQ Env -- fresh"
    objects = openshift.create(amqModels)

   }

   stage('Preparing JDG Dev Env') {
   
    echo "Deleting OLD JDG Env if exist"
    openshift.selector( 'all', [ application: cm.data['jdg-app-name'] ] ).delete()
    openshift.selector( 'pvc', [ application: cm.data['jdg-app-name'] ] ).delete()
    
    echo "Create New jdg Env -- fresh"
    objects = openshift.create(jdgModels)

   }

  }

 }
}
