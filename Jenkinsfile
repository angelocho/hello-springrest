pipeline {
    agent any
    options{
        timestamps()
        ansiColor('xterm')
    }
    stages {
        stage('Testing') {
            steps {
                     sh 'docker-compose config'
                     sh './gradlew test'
            }
            post {
                        always {
                                junit skipOldReports: true, skipPublishingChecks: true, testResults: 'build/test-results/test/*xml'
                        }       
                }
 
        }
        stage('BuildDocker') {
            steps {
                sh 'docker-compose build'
                sh 'git tag 1.0.${BUILD_NUMBER}'
                sshagent(['clave-sinensia']) {
                        sh 'git push --tags'
                }
                sh "docker tag ghcr.io/angelocho/hello-springrest/springrest:latest ghcr.io/angelocho/hello-springrest/springrest:1.0.${BUILD_NUMBER}"
            }
        }
        stage('Dockerlogin'){
           steps {
             withCredentials([string(credentialsId: 'github-token', variable: 'PAT')]) {
                 sh 'echo $PAT | docker login ghcr.io -u angelocho --password-stdin && docker-compose push && docker push ghcr.io/angelocho/hello-springrest/springrest:1.0.${BUILD_NUMBER}'

             }

           }
        }
        stage('BuildingElastic') {
            steps {
                withAWS(credentials:'clave-aws') {
                    dir('./elasticfolder') {
			sh 'eb create springrest-angelocho'
                    }   
                }
            }
        }
    }
}
