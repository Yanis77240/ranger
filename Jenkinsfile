pipeline {
    agent { 
        node {
            label 'docker-tdp-builder'
            }
      }
    environment {
        number="${currentBuild.number}"
      }
    triggers {
        pollSCM '0 1 * * *'
      }
    stages {
        stage('clone') {
            steps {
                echo "Cloning..."
                git branch: 'ranger-2.0-TDP-alliage', url: 'https://github.com/Yanis77240/ranger'
            }
        }
        stage('Build') {
            steps {
                echo "Building..."
                sh '''
                mvn clean install -DskipTests -Drat.numUnapprovedLicenses=1000
                '''
            }
        }
        /*stage('Test') {
            steps {
                echo "Testing..."
                sh '''
                mvn clean test -Drat.numUnapprovedLicenses=1000 --fail-never
                '''
            }
        }*/
        stage("Publish to Nexus Repository Manager") {
            steps {
                echo "Deploy..."
                withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh 'mvn clean deploy -DskipTests -Drat.numUnapprovedLicenses=1000 -s settings.xml'
                }
            }        
        }
        stage("Publish tar.gz to Nexus") {
            steps {
                echo "Deploy..."
                withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh '''
                    tar -cvzf ranger-0.9.1-TDP-0.1.0-SNAPSHOT.tar.gz target/*.tar.gz
                    curl -v -u $user:$pass --upload-file ranger-0.9.1-TDP-0.1.0-SNAPSHOT.tar.gz http://172.19.0.2:8081/repository/maven-tar-files/ranger/ranger-0.9.1-TDP-0.1.0-SNAPSHOT-${number}.tar.gz
                    '''
                }
            }        
        }
    }
}