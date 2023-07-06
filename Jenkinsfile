podTemplate(containers: [
    containerTemplate(
        name: 'tdp-builder', 
        image: 'yanisbariteau/tdp-builder:jenkins', 
        command: 'sleep', 
        args: '30d'
        )
  ]) {

    node(POD_LABEL) {
        container('tdp-builder') {
            environment {
                number="${currentBuild.number}"
            }
            stage('Git Clone') {
                echo "Cloning.."
                git branch: 'ranger-2.0-TDP-alliage-k8s', url: 'https://github.com/Yanis77240/ranger'
            }   
            stage ('Build') {
                echo "Building.."
                sh '''
                mvn clean install -DskipTests -Drat.numUnapprovedLicenses=1000
                '''
            }
            stage('Test') {
                echo "Testing.."
                sh '''
                mvn clean test -Drat.numUnapprovedLicenses=1000 --fail-never
                '''
            }
            stage('Deliver') {
                echo "Deploy..."
                withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh 'mvn clean deploy -DskipTests -Drat.numUnapprovedLicenses=1000 -s settings.xml'
                }
            }
            stage("Publish tar.gz to Nexus") {
                echo "Publish tar.gz..."
                withEnv(["number=${currentBuild.number}"]) {
                    withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                        sh '''
                        tar -cvzf ranger-0.9.1-TDP-0.1.0-SNAPSHOT.tar.gz target/*.tar.gz
                        curl -v -u $user:$pass --upload-file ranger-0.9.1-TDP-0.1.0-SNAPSHOT.tar.gz http://10.110.4.212:8081/repository/maven-tar-files/ranger/ranger-0.9.1-TDP-0.1.0-SNAPSHOT-${number}.tar.gz
                        '''
                    }
                }
            }       
        }
    }
}