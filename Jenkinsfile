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
        DOCKER_CRED_ID = "dockerhub"
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

        // ============================================================
        // 1. CLEAN WORKSPACE
        // ============================================================

        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        // ============================================================
        // 2. CHECKOUT CODE FROM GITHUB
        // ============================================================

        stage("Checkout from SCM") {
            steps {
                git branch: 'main',
                    credentialsId: 'github-token-auth',
                    url: 'https://github.com/hajera-nishat/CICD-App'
            }
        }

        // ============================================================
        // 3. BUILD & TEST
        // ============================================================

        stage("Build & Test Application") {
            steps {
                sh "mvn clean test package"
            }
        }

        // ============================================================
        // 4. SONARQUBE ANALYSIS
        // ============================================================

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        sh "mvn sonar:sonar -Dsonar.host.url=${SONAR_HOST_URL}"
                    }
                }
            }
        }

        // ============================================================
        // 5. QUALITY GATE
        // ============================================================

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate(abortPipeline: false)
                }
            }
        }

        // ============================================================
        // 6. ARTIFACTORY CONFIGURATION
        // ============================================================

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

        // ============================================================
        // 7. DEPLOY MAVEN ARTIFACT
        // ============================================================

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

        // ============================================================
        // 8. PUBLISH BUILD INFO
        // ============================================================

        stage("Publish Build Info") {
            steps {
                rtPublishBuildInfo(
                    serverId: "jfrog-server"
                )
            }
        }

        // ============================================================
        // 9. BUILD & PUSH DOCKER IMAGE
        // ============================================================

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

        // ============================================================
        // 10. TRIVY SECURITY SCAN
        // ============================================================

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

        // ============================================================
        // 11. CLEANUP LOCAL DOCKER IMAGES
        // ============================================================

        stage("Cleanup Docker Images") {
            steps {
                script {

                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
                    sh "docker rmi ${IMAGE_NAME}:latest || true"
                }
            }
        }

        // ============================================================
        // 12. DEPLOY TO AMAZON EKS
        // ============================================================

        stage("Deploy to Kubernetes") {
            steps {
                script {

                    withAWS(
                        credentials: "aws-eks",
                        region: "ap-south-2"
                    ) {

                        sh """
                            echo "======================================"
                            echo "AWS IDENTITY"
                            echo "======================================"

                            aws sts get-caller-identity

                            echo ""
                            echo "======================================"
                            echo "UPDATING EKS KUBECONFIG"
                            echo "======================================"

                            aws eks update-kubeconfig \
                                --region ap-south-2 \
                                --name cicd-eks

                            echo ""
                            echo "======================================"
                            echo "KUBERNETES NODES"
                            echo "======================================"

                            kubectl get nodes
                        """

                        dir("Kubernete") {

                            echo "Deploying Kubernetes Deployment..."

                            sh "kubectl apply -f regapp-deploy.yml"

                            echo "Deploying Kubernetes Service..."

                            sh "kubectl apply -f regapp-service.yml"

                            echo "Checking deployment rollout..."

                            sh """
                                kubectl rollout status \
                                deployment.apps/regapp-deployment \
                                --timeout=180s
                            """

                            echo "Checking pods..."

                            sh "kubectl get pods -o wide"

                            echo "Checking services..."

                            sh "kubectl get services"
                        }
                    }
                }
            }
        }
    }

    // ================================================================
    // POST BUILD EMAIL NOTIFICATIONS
    // ================================================================

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