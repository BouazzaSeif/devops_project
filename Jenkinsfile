pipeline {
    environment {
        EMAIL_RECIPIENTS = "seifeddine.bouazza@esprit.tn"
        registry = 'seifbouazza/devopsimage'
        registryCredential = 'dockerHub'
        dockerImage = ''
    }
    agent any
    tools {
        maven 'maven'
        jdk 'jdk'
    }
    stages {
        stage('cloning project') {
            steps {
                git 'https://github.com/BouazzaSeif/3ALINFO_05_G3.git'
            }
        }
        stage('build') {
            steps {
                bat 'mvn clean install -DskipTests -X '
            }
        }
        stage('Sonar') {
            steps {
                // bat 'mvn org.codehaus.mojo:sonar-maven-plugin:sonar -Dsonar.host.url=http://localhost:9000/'
                bat 'mvn sonar:sonar'
            }
        }
      /*  stage('Run test') {
           steps {
                 bat 'mvn test '
           }
       } */
        stage('Publish') {
            steps {
                nexusPublisher nexusInstanceId: 'localnexus', nexusRepositoryId: 'repo-snapshot-releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: 'target/timesheet_devops.jar']], mavenCoordinate: [artifactId: 'timesheet_devops', groupId: 'tn.esprit.spring', packaging: 'jar', version: '1.0']]]
        }
        }
       stage('Deploy on nexus') {
           steps {
              bat 'mvn deploy -Dmaven.test.skip=true'
           }
       }
        stage('Build Docker image') {
            steps { script { dockerImage = docker.build registry + ":$BUILD_NUMBER" } }
        }
        stage('Deploy Docker image') {
            steps
            {
                script
                 {
                    docker.withRegistry('', registryCredential)
                     {
                        dockerImage.push()
                     }
                 }
            }
        }
        stage('Cleaning up') {
            steps { bat "docker rmi $registry:$BUILD_NUMBER" }
        }
     }
    post {
            always {
                 emailext(
                        to: "${EMAIL_RECIPIENTS}",
                        replyTo: "${EMAIL_RECIPIENTS}",
                        subject:
                         "[BuildResult][${currentBuild.currentResult}] - Job '${env.JOB_NAME}' (${env.BUILD_NUMBER})",
                        mimeType: 'text/html',
                        body:
                        /* groovylint-disable-next-line GStringExpressionWithinString */
                        '''${JELLY_SCRIPT, template="custom-html.jelly"}'''
                        )
                deleteDir()
            }        
    }
    
     options {
            buildDiscarder(logRotator(numToKeepStr: '4'))
            timeout(time: 60, unit: 'MINUTES')
        }
}


