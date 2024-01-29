pipeline {
    agent any
    tools {
        jdk 'jdk-17'
        nodejs 'node-16'
    }
    environment {
        DOCKER_REGISTRY = 'batchusivaji'
        IMAGE_NAME = 'netflix'
        TMDB_API_CREDENTIALS = credentials('TMDB_V3_API_KEY')
        SONAR_SCANNER = tool 'sonar'
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('checkout branch') {
            steps {
                git branch: 'main', url: 'https://github.com/batchusivaji/netflix.git'
            }
        }
        stage('sonar-server') {
            steps {
                withSonarQubeEnv(installationName: 'sonar-server', credentialsId: 'SONAR_SEVER') {
                    sh "$SONAR_SCANNER/bin/sonar-scanner -Dsonar.projectName=jenkins-project-1 -Dsonar.projectKey=jenkins-project-1"
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'SONAR_SEVER'
                }
            }
        }
        stage("Install dependencies") {
            steps {
                script {
                    sh 'npm install'
                    sh 'node --version'
                    sh 'npm --version'
                }
            }
        }
        stage("OWASP Dependency-Check Scan") {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'Dp'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Trivy Filesystem Scan") {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }
        stage("docker image build && push") {
            steps {
                withCredentials([
                    string(credentialsId: 'DOCKER_PASSSWORD', variable: 'docker'),
                    string(credentialsId: 'TMDB_V3_API_KEY', variable: 'TMDB_V3_API_KEY')
                ]) {
                    echo "TMDB_V3_API_KEY: ${TMDB_V3_API_KEY}"
                    sh '''
                        docker login -u batchusivaji -p ${docker}
                        docker image build --build-arg TMDB_V3_API_KEY='${TMDB_V3_API_KEY}' \
                            -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${BUILD_ID} .
                    '''
                }
            }
        }
        stage("Trivy Image Scan"){
            steps{
                sh "trivy image ${DOCKER_REGISTRY}/${IMAGE_NAME}:${BUILD_ID} > trivyimage.txt "
                sh "docker image push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${BUILD_ID}"
                sh " docker image rmi ${DOCKER_REGISTRY}/${IMAGE_NAME}:${BUILD_ID}"

            }
        }
    }
}
