pipeline {
  agent any
  // This tools block tells Jenkins to add Maven to the PATH.
  tools {
    maven 'Maven-builder'
  }
  environment {
    // The name must match the one configured in Jenkins for SonarQube.
    SONARQUBE_SERVER = 'SonarQubeServer'
    // Additional environment variables (like credentials for Maven settings) can be added here.
  }
  stages {
    stage('Checkout') {
      steps {
        // Checkout the code from the release branch.
        // For a multibranch pipeline, this may already be done.
        git branch: 'release', url: 'https://github.com/DevSecOps-Stack/maven-unit-and-integration-tests.git'
      }
    }
    stage('Build & Test') {
      steps {
        // Run Maven build. With the tools directive, 'mvn' is available on the PATH.
        sh 'mvn clean install'
      }
    }
    stage('SonarQube Analysis') {
      steps {
        // Set up the environment with SonarQube configuration.
        withSonarQubeEnv(SONARQUBE_SERVER) {
          sh 'mvn sonar:sonar'
        }
      }
    }
    stage('Wait for Quality Gate') {
      steps {
        // Wait for the SonarQube Quality Gate result. Abort if it fails.
        timeout(time: 5, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
    stage('Deploy to Artifactory') {
      steps {
        // Use the Artifactory plugin’s rtMavenRun step to deploy artifacts.
        // Make sure the resolverId and deployerId match your global configuration.
        rtMavenRun(
          pom: 'pom.xml',
          goals: 'clean install deploy',
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
