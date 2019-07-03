#!groovy

pipeline {
  agent any
  stages {
   
    stage('Build') {
      steps {
      //  rocketSend channel: '#jenkins', message: 'Build Started'
        cleanWs()
        git changelog: false, poll: false, url: 'https://github.com/sushmithakumar/petclinic-new.git'
        sh '''
          start=`date +%s`
          set +x
          lastHash=`git log -n 1 --pretty=format:"%H"`
          lastOneHast=`git log -n 1 --skip 1 --pretty=format:"%H"`
          fileChanged=`git diff --name-only $lastHash $lastOneHast`
          message=`git log -1 --pretty=%B`
          author=` git log --format='%an <%ae>' ${lastHash} | head -1`
          echo "Commit Message: ${message}"
          echo "Changed files are: ${fileChanged}"
          echo "Commit Author: ${author}"
          end=`date +%s`
          runtime=$((end-start))
          echo "runtime is"
          echo $runtime
          '''
        sh ''' set -x
        mvn clean install -DskipTests -q

        #MSG=`git log -1 --pretty=%B`
        MSG=TEST-25

        echo "$MSG" >> msg.properties
        '''
        archiveArtifacts '**/*'
    //    rocketSend channel: '#general', message: 'Build Completed'
        //script {
        //  env.JIRA_NO = readFile 'msg.properties'
        //}
        //step([$class: 'JiraIssueUpdateBuilder', comment: 'Build Completed by jenkins', jqlSearch: "project=TEST and key=${env.JIRA_NO}", workflowActionName: 'BUILD'])
      }
    }
    
    stage('Unit Test') {
      steps {
        sh '''
        set +x
        mvn clean test -q'''
        //script {
        //  env.JIRA_NO = readFile 'msg.properties'
        //}
        sh "echo From Readfile ${env.JIRA_NO}"
       // rocketSend channel: '#general', message: 'Unit Test Completed'
    //    step([$class: 'JiraIssueUpdateBuilder', comment: 'Unit Testing completed by jenkins', jqlSearch: "project=TEST and key=${env.JIRA_NO}", workflowActionName: 'UNIT TESTING'])
      }
    }
    
    stage('Nexus Upload') {
      steps {
        withCredentials([usernameColonPassword(credentialsId: 'ethanadmin-creds', variable: 'nexus')]){
	  sh '''
          set +x
          mvn clean install -DskipTests -q
          curl -v -u $nexus --upload-file ${WORKSPACE}/target/petclinic.war http://nexus:8083/nexus/repository/petclinic/
	  echo "Uploaded Artefact" 
	  '''
	  }
	 // rocketSend channel: '#general', message: 'Uploaded Artefact to Nexus'
	  }
	  }
	  
	stage('Sonar Analysis') {
      steps {
        withSonarQubeEnv('SonarQube') {
          sh '''
          set +x
          mvn sonar:sonar -q
          echo "Sonar Stat"
          curl --silent -u 63284ac002d928944d813ea49571daa7cda7f2c2: "http://sonar.ethan.svc.cluster.local:9001/sonar/api/resources?resource=org.springframework.samples:spring-petclinic&metrics=lines,blocker_violations,ncloc,classes,directories,files,functions,projects,public_api,statements,branch_coverage"'''
        } 
	  }
	}
  }

}
