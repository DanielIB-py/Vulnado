pipeline {
    agent { label "Jenkins-Agent" }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
            APP_NAME = "vulnado-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "daniel217x"
            DOCKER_PASS = 'dockerhub'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
            BRIDGE_POLARIS_APPLICATION_NAME = "WHIPSO"
            BRIDGE_POLARIS_PROJECT_NAME = "Test Vulnapipeline"
          
    }

    

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
               steps {
                   git branch: 'master', credentialsId: 'github', url: 'https://github.com/Danielib217/vulnado'
               }
        }

      stage("Build Application") {
               steps {
                   sh "mvn clean package"
               }
        }

      stage("Test Application") {
               steps {
                   sh "mvn test"
               }
        }
        
      stage("SonarQube Analysis"){
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token'){
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }
       
    stage('polaris') {
        steps {
            
                script {
                    withCredentials([string(credentialsId: 'poc.polarissynopsys.com', variable: 'BRIDGE_POLARIS_ACCESSTOKEN')]) {
            
                        sh('curl -fLsS -o bridge.zip $BRIDGECLI_LINUX64 && unzip $RUNNER_TEMP bridge.zip && rm -f bridge.zip $WORKSPACE_TMP/synopsys-bridge --verbose && export BRIDGE_POLARIS_ACCESSTOKEN=apf9q719j52cpdso23iookbodug2uq180jtki1s5kc4u7bp9g78qi0g6odfqrbm2kk7a6t2g9jrqe synopsys-bridge --stage polaris polaris.project.name="Test Vulna" && polaris.application.name="WHIPSO" && polaris.assessment.types=SAST,SCA && polaris.serverurl="https://poc.polaris.synopsys.com/"')
                    
                     }
                }
            }
         }
                           
         
        stage("Build & Push Docker Image"){
            steps{
                script{
                    docker.withRegistry('',DOCKER_PASS){
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                    docker.withRegistry('',DOCKER_PASS){
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }
        stage("Trivy Scan"){
            steps{
                script{
                    sh('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image daniel217x/vulnado-pipeline:latest --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
            }
        }

        stage('Cleanup Artifacts'){
            steps{
                script{
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }
    }
}
