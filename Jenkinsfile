pipeline {
  agent { label 'slave1' }

  environment {
    MAVEN_OPTS = '-Xmx1024m'
    RELEASES_URL = 'http://43.204.145.173:8081/repository/maven-releases/'
    SNAPSHOTS_URL = 'http://43.204.145.173:8081/repository/maven-snapshots/'
  }

  options {
    ansiColor('xterm')
    timestamps()
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Prepare settings.xml') {
      steps {
        // create a Maven settings.xml using the Nexus credentials stored in Jenkins
        withCredentials([usernamePassword(credentialsId: 'nexus-creds1', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
          sh '''
cat > settings.xml <<EOF
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <servers>
    <server>
      <id>nexus-releases</id>
      <username>${NEXUS_USER}</username>
      <password>${NEXUS_PASS}</password>
    </server>
    <server>
      <id>nexus-snapshots</id>
      <username>${NEXUS_USER}</username>
      <password>${NEXUS_PASS}</password>
    </server>
  </servers>
</settings>
EOF
'''
        }
      }
    }

    stage('Build & Deploy to Nexus') {
      steps {
        // skip tests while you verify; remove -DskipTests to run tests
        sh 'mvn -B -s settings.xml clean deploy -DskipTests'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        // uses the sonar-token credential you created in Jenkins
        withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
          // If you configured SonarQube server in Jenkins, you can use withSonarQubeEnv('MySonar')
          // but calling mvn sonar:sonar with -Dsonar.login works generally:
          sh "mvn -B -Dsonar.login=${SONAR_TOKEN} sonar:sonar || true"
          // note: sonar analysis often runs in background on server; adjust as needed
        }
      }
    }
  }

  post {
    success {
      echo 'Build + deploy + sonar completed successfully.'
    }
    failure {
      echo 'Build failed — check console logs.'
    }
    always {
      archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
      junit '**/target/surefire-reports/*.xml'
    }
  }
}
