pipeline {
  agent any
  environment {
    // The name must match the one configured in Jenkins for SonarQube.
    SONARQUBE_SERVER = 'SonarQubeServer'
    // Maven settings can be passed through if you have configured a Managed File.
    // For example: MAVEN_SETTINGS = credentials('MAVEN_SETTINGS_ID')
  }
  stages {
    stage('Checkout') {
      steps {
        // Checkout the code from the release branch.
        // If using a multibranch pipeline, Jenkins will already check out the correct branch.
        // Otherwise, specify the branch:
        git branch: 'release', url: 'https://github.com/DevSecOps-Stack/maven-unit-and-integration-tests.git'
      }
    }
    stage('Build & Test') {
      steps {
        // Run Maven to compile, run unit tests, and run integration tests.
        // Adjust the Maven goals if necessary.
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
        // Wait for SonarQube Quality Gate result.
        // If the Quality Gate fails, the pipeline will abort.
        timeout(time: 5, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
    stage('Deploy to Artifactory') {
      steps {
        // Use the Artifactory plugin’s step to run Maven with deployment.
        // Ensure that the resolver and deployer IDs match your global Jenkins configuration.
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
