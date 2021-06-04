try{
    node{
        def mavenHome
        def mavenCMD
        def docker
        def dockerCMD
        def tagName = "2.0"
        
        stage('Preparation'){
            echo "Preparing the Jenkins environment with required tools for pipeline"
            mavenHome = tool name: 'maven', type: 'maven'
            mavenCMD = "${mavenHome}/bin/mvn"
            docker = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
            dockerCMD = "$docker/bin/docker"
        }
        
        stage('Checkout the GIT SCM'){
            echo "Checking out the code from Git Repository"
            git 'https://github.com/ankit8917/casestudy_devops10.git'
        }
        stage('Build, Test and package using Maven Tool'){
            echo "Building, Testing and Packaging the application"
            sh "${mavenCMD} clean package"
        }
        stage('Code analysis by Sonarqube Scanner'){
            echo "Scanning application for vulnerabilities"
            sh "${mavenCMD} sonar:sonar -Dsonar.projectKey=org.test:my-test-app -Dsonar.host.url=http://3.93.148.87:9000 -Dsonar.login=886273ce0797a98587ae49ded273863a196a9a68"
        }
        stage('Junit Test Cases'){
            echo "Generating Junit Test Report.."
            sh "${mavenCMD} test site"
        }
        stage('HTML Reports for Junit Test Cases..!'){
            echo "Publishing HTML Reports for Junit Test Cases"
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'target/site/', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: ''])
        }    
        stage('Building Docker Image') {
            echo "Building Docker Image for spring Application"
            sh "${dockerCMD} build -t ankit8917/my-jar-app:${tagName} ."
        }  
        stage('Push Docker Image to Dockerhub') {
            withCredentials([string(credentialsId: 'dockerhubp2', variable: 'dockerhubp1')]) {
                sh "${dockerCMD} login -u ankit8917 -p ${dockerhubp1}"
                sh "${dockerCMD} push ankit8917/my-jar-app:${tagName}"
            }
        }    
        stage('Configure Worker node and Deploy Application'){
            echo "Configure Worker node and Deploy Application using Ansible"
            ansiblePlaybook credentialsId: 'ansible-playbook', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: 'playbook.yaml'
        }
        stage('Clean Up'){
           echo "Cleaning up the Workspace of case-study-pipeline"
            cleanWs()    
            
        }  
    }
}    
catch(Exception err){
    echo "Exception occured..."
    currentBuild.result="FAILURE"
    emailext body: 'Your build has been failed.', subject: 'Case Study Pipeline Result', to: 'ankit8917'
}
finally {
    (currentBuild.result!= "ABORTED") && node("master") {
        echo "Finally gets executed and Email notification sent for every build"
        emailext body: 'Pipeline Execution Went fine', subject: 'Case Study Pipeline Result', to: 'ankit8917'
    }
    
}
        
