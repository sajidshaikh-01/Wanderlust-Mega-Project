@Library('Shared') _
pipeline {
    agent any

    environment {
        SONAR_HOME = tool "Sonar"     // SonarQube Scanner tool name in Jenkins
    }

    parameters {
        string(name: 'FRONTEND_DOCKER_TAG', defaultValue: '', description: 'Docker tag for frontend')
        string(name: 'BACKEND_DOCKER_TAG', defaultValue: '', description: 'Docker tag for backend')
    }

    stages {
        stage("Validate Parameters") {
            steps {
                script {
                    if (params.FRONTEND_DOCKER_TAG == '' || params.BACKEND_DOCKER_TAG == '') {
                        error("❌ Both FRONTEND_DOCKER_TAG and BACKEND_DOCKER_TAG must be provided.")
                    }
                }
            }
        }

        stage("Workspace Cleanup") {
            steps {
                cleanWs()
            }
        }

        stage("Git: Checkout Code") {
            steps {
                script {
                    code_checkout("https://github.com/sajidshaikh-01/Wanderlust-Mega-Project.git", "main")
                }
            }
        }

        stage("Trivy: Filesystem Scan") {
            steps {
                script {
                    trivy_scan()
                }
            }
        }

        stage("OWASP: Dependency Check") {
            steps {
                script {
                    owasp_dependency()
                }
            }
        }

        stage("SonarQube: Code Analysis") {
            steps {
                script {
                    sonarqube_analysis("Sonar", "wanderlust", "wanderlust")
                }
            }
        }

        stage("SonarQube: Quality Gate") {
            steps {
                script {
                    sonarqube_code_quality()
                }
            }
        }

        stage("Exporting Environment Variables") {
            parallel {
                stage("Backend Env Setup") {
                    steps {
                        dir("Automations") {
                            sh "bash updatebackendnew.sh"
                        }
                    }
                }

                stage("Frontend Env Setup") {
                    steps {
                        dir("Automations") {
                            sh "bash updatefrontendnew.sh"
                        }
                    }
                }
            }
        }

        stage("Docker: Build Images") {
            steps {
                script {
                    dir('backend') {
                        docker_build("wanderlust-backend-beta", "${params.BACKEND_DOCKER_TAG}", "sajid0100")
                    }

                    dir('frontend') {
                        docker_build("wanderlust-frontend-beta", "${params.FRONTEND_DOCKER_TAG}", "sajid0100")
                    }
                }
            }
        }

        stage("Docker: Push to DockerHub") {
            steps {
                script {
                    docker_push("wanderlust-backend-beta", "${params.BACKEND_DOCKER_TAG}", "sajid0100")
                    docker_push("wanderlust-frontend-beta", "${params.FRONTEND_DOCKER_TAG}", "sajid0100")
                }
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: '**/*.xml', followSymlinks: false
            echo "✅ CI Pipeline completed successfully. Triggering CD job..."

            build job: "Wanderlust-CD", parameters: [
                string(name: 'FRONTEND_DOCKER_TAG', value: "${params.FRONTEND_DOCKER_TAG}"),
                string(name: 'BACKEND_DOCKER_TAG', value: "${params.BACKEND_DOCKER_TAG}")
            ]
        }
        failure {
            echo "❌ CI Pipeline failed. Please check logs."
        }
    }
}
