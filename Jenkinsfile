pipeline {
    agent any
    //{
        // docker {
        //    image 'maven:3.9.0'
        //    args '-v /root/.m2:/root/.m2'
        //}
    //}

    environment {
        APPLICATION     = 'dependency-track'
        APP_NAME        = "dvwa-spring"
        APP_VERSION     = "1"
        DEFAULT_BRANCH  = 'master'
        USER_ID         = sh (returnStdout: true, script: 'echo \$(git show -s --pretty=%an)').trim()
    }

    stages {
        stage('Build') {
            steps {
                sh './mvnw -B -DskipTests clean package'
            }
        }
        stage('Test') {
            steps {
                sh './mvnw test'
            }
        }

        stage ('Dependency track') {
          steps {
            // First generate Bill of Materials
            sh './mvnw org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom'
            // Then ingest results to dependency track platform
            withCredentials([string(credentialsId: 'dependency-track-api-key-global', variable: 'API_KEY')])
            {
              dependencyTrackPublisher artifact:'target/bom.xml', projectName:"${env.APP_NAME}", projectVersion:"${env.APP_VERSION}", synchronous:true, dependencyTrackApiKey:"${API_KEY}"
             }
          }
        }
    }
}
