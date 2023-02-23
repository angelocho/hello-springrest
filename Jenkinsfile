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
                     sh './gradlew check'
            }
            post {
                        always {
                                junit(testResults: 'build/test-results/test/*xml', allowEmptyResults: true)
                                jacoco classPattern: 'build/classes/java/main', execPattern: 'build/jacoco/*.exec', sourcePattern: 'src/main/java/com/example/restservice'
                                recordIssues(tools: [pmdParser(pattern: 'build/reports/pmd/*.xml')])
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
			sh 'eb deploy springrest-angelocho'
                    }   
                }
            }
        }
    }
}
