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
        // Checkout the code from the release branch.
        git branch: 'release', url: 'https://github.com/DevSecOps-Stack/maven-unit-and-integration-tests.git'
      }
    }
    stage('Build & Test') {
      steps {
        // Build and run tests with Maven, using a local repository in the workspace.
        sh 'mvn clean install -Dmaven.repo.local=${WORKSPACE}/.m2/repository'
      }
    }
    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv(SONARQUBE_SERVER) {
          // Run SonarQube analysis while overriding user.home and sonar.working.directory to the workspace.
          // Also exclude Terraform files from analysis to avoid triggering the problematic sensor.
          sh 'mvn sonar:sonar -Duser.home=${WORKSPACE} -Dsonar.working.directory=${WORKSPACE} -Dmaven.repo.local=${WORKSPACE}/.m2/repository -Dsonar.exclusions=**/*.tf,**/*.tfvars,**/*.tfstate'
        }
      }
    }
    stage('Wait for Quality Gate') {
      steps {
        // Increase timeout if necessary.
        timeout(time: 10, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
    stage('Deploy to Artifactory') {
      steps {
        // Use the Artifactory plugin's rtMavenRun step to deploy the artifact.
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
