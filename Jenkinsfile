pipeline {
    agent any

    tools {
        maven 'mymaven'
    }
    environment {
        IMAGE = 'pet-clinic-image'
        SONAR_SCANNER_HOME = tool 'sonarscanner'
        SONAR_TOKEN = credentials('SONAR_TOKEN')
        DOCKER_USER = 'sukruth17'
        DOCKER_PASS = credentials('dockerhub-pass')
    }
    
    stages {
        stage("Building source code") {
            steps {
                echo "Building ...."
                sh "mvn -B -DskipTests clean package"
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        stage("Unit Test of source code") {
            steps {
                echo "Testing....."
                sh "mvn test -Dcheckstyle.skip"
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage("Code analysis") {
            steps {
                sh "mvn checkstyle:checkstyle"
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(installationName: 'sonarscanner') {
                    sh "${SONAR_SCANNER_HOME}/bin/sonar-scanner " +
                        "-Dsonar.projectKey=sukruth-gms_PetClinic-Build " +
                        "-Dsonar.organization=sukruth-gms-sonar " +
                        "-Dsonar.sources=src/main " +
                        "-Dsonar.exclusions=**/*.java " +
                        "-Dsonar.host.url=https://sonarcloud.io " +
                        "-Dsonar.token=${SONAR_TOKEN} " +
                        "-Dsonar.junit.reportsPath=target/surefire-reports/ " +
                        "-Dsonar.jacoco.reportsPath=target/jacoco.exec " +
                        "-Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml"
                }
            }
        }

        stage('Build Image') {
            steps {
                echo "Building Docker IMAGE"
                script {
                    def BUILD_NUMBER = env.BUILD_NUMBER
                    echo "Building Docker IMAGE with build number: $BUILD_NUMBER"
                    sh "docker build -t $IMAGE:$BUILD_NUMBER ."
                }
            }
        }

        stage('Push Image') {
            steps {
                echo 'Pushing to Dockerhub...'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-pass', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo 'Logging in ****'
                        docker login -u $DOCKER_USER -p $DOCKER_PASS
                        echo '*** Tagging image ***'
                        docker tag $IMAGE:$BUILD_NUMBER $DOCKER_USER/$IMAGE:$BUILD_NUMBER
                        echo '*** Pushing image ***' 
                        docker push $DOCKER_USER/$IMAGE:$BUILD_NUMBER
                    """
                }
            }
        }
    }
}
