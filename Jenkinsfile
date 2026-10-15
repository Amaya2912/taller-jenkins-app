
pipeline {
    agent none

    environment {
        IMAGEN = "taller-app:${env.BUILD_NUMBER}"
        CONTENEDOR = "taller-app-${env.BUILD_NUMBER}"
    }

    stages {
        stage('Compilar') {
            agent {
                docker {
                    image 'maven:3.9-eclipse-temurin-21'
                    args '--network host'
                    reuseNode true
                }
            }

            environment {
                HOME = "${env.WORKSPACE}"
                MVN = 'mvn -B -Dmaven.repo.local=.m2/repository'
            }

            steps {
                sh '$MVN compile'
            }
        }

        stage('Pruebas') {
            agent {
                docker {
                    image 'maven:3.9-eclipse-temurin-21'
                    args '--network host'
                    reuseNode true
                }
            }

            environment {
                HOME = "${env.WORKSPACE}"
                MVN = 'mvn -B -Dmaven.repo.local=.m2/repository'
            }

            steps {
                sh '$MVN test'
            }

            post {
                always {
                    junit allowEmptyResults: true,
                          testResults: 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Empaquetar') {
            agent {
                docker {
                    image 'maven:3.9-eclipse-temurin-21'
                    args '--network host'
                    reuseNode true
                }
            }

            environment {
                HOME = "${env.WORKSPACE}"
                MVN = 'mvn -B -Dmaven.repo.local=.m2/repository'
            }

            steps {
                sh '$MVN -DskipTests package'

                archiveArtifacts artifacts: 'target/taller-app.jar',
                                 fingerprint: true
            }
        }

        stage('Construir imagen') {
            agent any

            steps {
                sh 'docker build --network host -t "$IMAGEN" .'
            }
        }

        stage('Smoke test') {
            agent any

            steps {
                sh 'docker run -d --name "$CONTENEDOR" -p 18000:8000 "$IMAGEN"'

                sh '''
                    curl --fail \
                         --retry 10 \
                         --retry-connrefused \
                         --retry-delay 2 \
                         http://docker:18000/health
                '''
            }

            
            post {
                always {
                    sh 'docker rm -f "$CONTENEDOR" || true'
                }
            }
        }

        
        stage('Desplegar') {
            agent any

            steps {
                sh '''
                    docker rm -f taller-app || true
                    docker run -d --name taller-app --restart unless-stopped -p 8000:8000 "$IMAGEN"
                    sleep 5
                    curl -fs http://docker:8000/version
                '''
            }
        }

    }
}
