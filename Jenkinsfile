pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"

        // Docker Hub
        DOCKER_USER = "hajeranishat11"
        DOCKER_CRED_ID = "docker-hub"
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"

        // SonarQube
        SONAR_HOST_URL = "http://172.31.3.255:9000"

        // JFrog Artifactory
        JFROG_URL = "http://172.31.3.255:8081/artifactory"

        // Jenkins email notifications
        NOTIFICATION_EMAIL = "hajeranishat11@gmail.com"
    }

    stages {

        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main',
                    credentialsId: 'github-token-auth',
                    url: 'https://github.com/hajera-nishat/CICD-App'
            }
        }

        stage("Build & Test Application") {
            steps {
                sh "mvn clean test package"
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'SonarQube-token') {
                        sh "mvn sonar:sonar -Dsonar.host.url=${SONAR_HOST_URL}"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate(
                        abortPipeline: false,
                        credentialsId: 'SonarQube-token'
                    )
                }
            }
        }

        stage("Artifactory Configuration") {
            steps {

                rtServer(
                    id: "jfrog-server",
                    url: "${JFROG_URL}",
                    credentialsId: "jfrog"
                )

                rtMavenDeployer(
                    id: "MAVEN_DEPLOYER",
                    serverId: "jfrog-server",
                    releaseRepo: "libs-release-local",
                    snapshotRepo: "libs-snapshot-local"
                )

                rtMavenResolver(
                    id: "MAVEN_RESOLVER",
                    serverId: "jfrog-server",
                    releaseRepo: "libs-release",
                    snapshotRepo: "libs-snapshot"
                )
            }
        }

        stage("Deploy Artifacts") {
            steps {
                rtMavenRun(
                    tool: "Maven",
                    pom: "webapp/pom.xml",
                    goals: "clean install",
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
            }
        }

        stage("Publish Build Info") {
            steps {
                rtPublishBuildInfo(
                    serverId: "jfrog-server"
                )
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry(
                        "https://index.docker.io/v1/",
                        DOCKER_CRED_ID
                    ) {
                        def docker_image = docker.build(
                            "${IMAGE_NAME}:${IMAGE_TAG}"
                        )

                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push("latest")
                    }
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                script {
                    sh """
                        docker run --rm \
                        -v /var/run/docker.sock:/var/run/docker.sock \
                        aquasec/trivy image \
                        ${IMAGE_NAME}:latest \
                        --no-progress \
                        --scanners vuln \
                        --exit-code 0 \
                        --severity HIGH,CRITICAL \
                        --format table
                    """
                }
            }
        }

        stage("Cleanup Docker Images") {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
                    sh "docker rmi ${IMAGE_NAME}:latest || true"
                }
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                script {
                    dir("Kubernete") {

                        kubeconfig(
                            credentialsId: "kubernetes",
                            serverUrl: ""
                        ) {

                            sh "kubectl apply -f regapp-deploy.yml"
                            sh "kubectl apply -f regapp-service.yml"

                            sh """
                                kubectl rollout restart \
                                deployment.apps/registerapp-deployment
                            """
                        }
                    }
                }
            }
        }
    }

    post {

        failure {
            emailext(
                body: '''${SCRIPT, template="groovy-html.template"}''',
                subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - Failed",
                mimeType: "text/html",
                to: "${NOTIFICATION_EMAIL}"
            )
        }

        success {
            emailext(
                body: '''${SCRIPT, template="groovy-html.template"}''',
                subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - Successful",
                mimeType: "text/html",
                to: "${NOTIFICATION_EMAIL}"
            )
        }
    }
}