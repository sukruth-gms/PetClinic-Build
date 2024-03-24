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
        ARTIFACTORY_URL = 'https://appartifactjfrog.jfrog.io/'
        ARTIFACTORY_REPO = 'artifactstore-maven-remote'
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
                script {
                    def server = Artifactory.server("${ARTIFACTORY_URL}")
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "target/*.jar",
                                "target": "$ARTIFACTORY_REPO/"
                            }
                        ]
                    }"""
                    server.upload(uploadSpec)
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
                    script {
                        echo 'Logging in ****'
                        sh "docker login -u $DOCKER_USER -p $DOCKER_PASS"
                        echo '*** Tagging image ***'
                        sh "docker tag $IMAGE:$BUILD_NUMBER $DOCKER_USER/$IMAGE:$BUILD_NUMBER"
                        echo '*** Pushing image ***' 
                        sh "docker push $DOCKER_USER/$IMAGE:$BUILD_NUMBER"
                    }
                }
            }
        }
    }
}
