pipeline {
  agent any
  // any, none, label, node, docker, dockerfile, kubernetes

  tools{
    maven 'my_maven'
  }

  environment {
    gitName = 'oolr'
    gitEmail = 'jyy013@gmail.com'
    gitWebaddress = 'https://github.com/oolr/sb_code2.git'
    gitSshaddress = 'git@github.com:oolr/sb_code2.git'
    gitCredential = 'git_cre' //github credential 생성시의 ID
    dockerHubRegistry = 'jyy0103/sbimage'
    dockerHubRegistry2 = 'jyy0103/uparupa'
    dockerHubRegistryCredential = 'docker_cre'
  }


  stages {
    stage('Checkout Github') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: gitCredential, url: gitWebaddress]]])
      }
      post {
        failure {
            echo 'Repository clone failure'
        }
        success {
            echo 'Repository clone success'
        }
     }
    }

    stage('Maven Build') {
      steps {
        sh 'mvn clean install'
        // maven 플러그인이 미리 설치 되어 있어야함
      }
      post {
        failure {
            echo 'Maven build failure'
        }
        success {
            echo 'Maven build success'
        }
     }
    }

    stage('Docker image Build') {
      steps {
        sh "docker build -t ${dockerHubRegistry}:${currentBuild.number} ."
        sh "docker build -t ${dockerHubRegistry}:latest ."
        
        sh "docker build -t ${dockerHubRegistry2}:${currentBuild.number} -f m-Dockerfile ."
        sh "docker build -t ${dockerHubRegistry2}:latest ."
        // jyy01-3/sbimage:4  이런식으로 빌드가 될 것이다.
        // currentBuild.number 젠킨스에서 제공하는 빌드넘버변수.
      }
      post {
        failure {
            echo 'Docker image build failure'
        }
        success {
            echo 'Docker image build success'
        }
     }
    }

    stage('Docker image Push') {
      steps {
        withDockerRegistry(credentialsId: dockerHubRegistryCredential, url: '') {
          // withDockerRegistry : docker pipeline 플러그인 설치시 사용가능.
          // dockerHubRegistryCredential : environment에서 선언한 docker_cre  
            sh "docker push ${dockerHubRegistry}:${currentBuild.number}"
            sh "docker push ${dockerHubRegistry}:latest"

            sh "docker push ${dockerHubRegistry2}:${currentBuild.number}"
            sh "docker push ${dockerHubRegistry2}:latest"
          }
      }
      post {
        failure {
            echo 'Docker image push failure'
            sh "docker image rm -f ${dockerHubRegistry}:${currentBuild.number}"
            sh "docker image rm -f ${dockerHubRegistry}:latest"
          
            sh "docker image rm -f ${dockerHubRegistry2}:${currentBuild.number}"
            sh "docker image rm -f ${dockerHubRegistry2}:latest"
        }
        success {            
            sh "docker image rm -f ${dockerHubRegistry}:${currentBuild.number}"
            sh "docker image rm -f ${dockerHubRegistry}:latest"
          
            sh "docker image rm -f ${dockerHubRegistry2}:${currentBuild.number}"
            sh "docker image rm -f ${dockerHubRegistry2}:latest"
            echo 'Docker image push success'
        }
     }
    }

      stage('k8s manifest file update') {
      steps {
        git credentialsId: gitCredential,
            url: gitWebaddress,
            branch: 'main'
        
        // 이미지 태그 변경 후 메인 브랜치에 푸시
        sh "git config --global user.email ${gitEmail}"
        sh "git config --global user.name ${gitName}"
        sh "sed -i 's@${dockerHubRegistry}:.*@${dockerHubRegistry}:${currentBuild.number}@g' deploy/sb-deploy.yml"
        sh "sed -i 's@${dockerHubRegistry2}:.*@${dockerHubRegistry2}:${currentBuild.number}@g' deploy/my-deploy.yml"
        sh "git add ."
        sh "git commit -m 'fix:${dockerHubRegistry} ${dockerHubRegistry2} ${currentBuild.number} image versioning'"
        sh "git branch -M main"
        sh "git remote remove origin"
        sh "git remote add origin ${gitSshaddress}"
        sh "git push -u origin main"

      }
      post {
        failure {
          echo 'k8s manifest file update failure'
        }
        success {
          echo 'k8s manifest file update success'  
        }
      }
    }


  }
}
