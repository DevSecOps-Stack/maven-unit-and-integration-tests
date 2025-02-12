pipeline {
  agent any
  tools {
    maven 'maven-builder'
  }
  environment {
    SONARQUBE_SERVER = 'SonarQubeServer'  // Must match Jenkins' configured server name
    SONAR_HOST_URL = 'http://sonar-route-sonar.apps.openshift-cluster.softekh.com'  // Explicit URL
    SONAR_AUTH_TOKEN = credentials('sonar-auth-token-id')  // Jenkins credential ID for SonarQube token
    HOME = "${WORKSPACE}"
    SONAR_USER_HOME = "${WORKSPACE}/.sonar"
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
          sh '''
            mkdir -p ${WORKSPACE}/.sonar/cache
            mvn sonar:sonar \
              -Dsonar.host.url=${SONAR_HOST_URL} \
              -Dsonar.login=${SONAR_AUTH_TOKEN} \
              -Dsonar.working.directory=${WORKSPACE} \
              -Dsonar.userHome=${WORKSPACE}/.sonar \
              -Dmaven.repo.local=${WORKSPACE}/.m2/repository \
              -Dsonar.exclusions=**/*.tf,**/*.tfvars,**/*.tfstate \
              -Dsonar.iac.terraform.enabled=false \
              -Dsonar.iac.enabled=false
          '''
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
      echo 'Build, tests, and SonarQube analysis succeeded!'
    }
    failure {
      echo 'Pipeline failed. Check logs for details.'
    }
  }
}
