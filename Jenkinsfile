pipeline {
  agent any
  tools {
    maven 'maven-builder'
  }
  environment {
    // The name must match the one configured in Jenkins for SonarQube.
    SONARQUBE_SERVER = 'SonarQubeServer'
    // Set HOME to the workspace so that SonarQube can create its cache there.
    HOME = "${WORKSPACE}"
  }
  stages {
    stage('Checkout') {
      steps {
        // Checkout the code from the release branch.
        git branch: 'release', url: 'https://github.com/DevSecOps-Stack/maven-unit-and-integration-tests.git'
      }
    }
    stage('Build & Test') {
      steps {
        // Use a custom local repository inside the workspace.
        sh 'mvn clean install -Dmaven.repo.local=${WORKSPACE}/.m2/repository'
      }
    }
    stage('SonarQube Analysis') {
      steps {
        // Use withSonarQubeEnv to configure SonarQube settings.
        withSonarQubeEnv(SONARQUBE_SERVER) {
          // Override user.home to force the Sonar Scanner to use a writable directory.
          sh 'mvn sonar:sonar -Duser.home=${WORKSPACE} -Dmaven.repo.local=${WORKSPACE}/.m2/repository'
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
        // Use the Artifactory plugin’s rtMavenRun step to deploy the artifact.
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
