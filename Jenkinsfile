pipeline {
  agent any
  tools {
    maven 'maven-builder'
  }
  environment {
    SONARQUBE_SERVER = 'SonarQubeServer'
    HOME = "${WORKSPACE}"
  }
  stages {
    stage('Checkout') {
      steps {
        git branch: 'release', url: 'https://github.com/DevSecOps-Stack/maven-unit-and-integration-tests.git'
      }
    }
    stage('Build & Test') {
      steps {
        sh 'mvn clean install -Dmaven.repo.local=${WORKSPACE}/.m2/repository'
      }
    }
  stage('SonarQube Analysis') {
  steps {
    withSonarQubeEnv(SONARQUBE_SERVER) {
      sh 'mvn sonar:sonar -Duser.home=${WORKSPACE} ' +
         '-Dsonar.working.directory=${WORKSPACE} ' +
         '-Dmaven.repo.local=${WORKSPACE}/.m2/repository ' +
         '-Dsonar.exclusions=**/*.tf,**/*.tfvars,**/*.tfstate ' +
         '-Dsonar.iac.terraform.enabled=false ' +
         '-Dsonar.sensors=-Terraform'
      }
    } 
  }
    stage('Wait for Quality Gate') {
      steps {
        timeout(time: 5, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
    stage('Deploy to Artifactory') {
      steps {
        rtMavenRun(
          pom: 'pom.xml',
          goals: 'clean install deploy -Dmaven.repo.local=${WORKSPACE}/.m2/repository',
          resolverId: 'ARTIFACTORY_RESOLVER',
          deployerId: 'ARTIFACTORY_DEPLOYER'
        )
      }
    }
  }
  post {
    success {
      echo 'Build, tests, SonarQube analysis, and deployment succeeded!'
    }
    failure {
      echo 'One or more steps failed. Check the logs for details.'
    }
  }
}
