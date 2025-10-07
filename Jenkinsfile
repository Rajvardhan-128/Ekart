// ---------------- New pipeline with disable owasp -----------------------------------
pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        // Temporarily skipping NVD_API_KEY since OWASP scan is disabled for now
        // NVD_API_KEY = credentials('nvd-api-key')
    }

    tools {
        maven 'maven3'
        jdk 'jdk-17'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Rajvardhan-128/Ekart.git'
            }
        }

     stage('OWASP Dependency Check') {
    steps {
        withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
            dependencyCheck additionalArguments: "--nvdApiKey=$NVD_API_KEY",
                            odcInstallation: 'DC'
        }
    }
}
   
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Unit Tests') {
            steps {
                // You had -DskipTests=true, which skips tests.
                // If you want to run tests, remove that flag.
                sh "mvn test -DskipTests=true"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-token') {
                    sh """
                        ${env.SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=EKART \
                        -Dsonar.projectName=EKART \
                        -Dsonar.java.binaries=target/classes
                    """
                }
            }
        }

        // -------------------------------------------------------
        // 🔧 OWASP Dependency Check temporarily disabled
        // -------------------------------------------------------
        /*
        stage('OWASP Dependency Check') {
            steps {
                withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
                    dependencyCheck additionalArguments: "--nvdApiKey=$NVD_API_KEY",
                                    odcInstallation: 'DC'
                }
            }
        }
        */
        // -------------------------------------------------------

        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }

        stage('Deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk-17', maven: 'maven3', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }

        stage('Build and Tag Docker Image') {
            steps {
                script {
                    sh "docker build -t rock284/ekart:latest -f docker/Dockerfile ."
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'dockerhub-pwd', variable: 'dockerhubpwd')]) {
                        sh 'docker login -u rock284 -p ${dockerhubpwd}'
                    }
                    sh 'docker push rock284/ekart:latest'
                }
            }
        }

        stage('EKS and Kubectl Configuration') {
            steps {
                script {
                    sh 'aws eks update-kubeconfig --region ap-south-1 --name project-cluster'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh 'kubectl apply -f deploymentservice.yml'
                }
            }
        }
    }

    post {
        always {
            echo "✅ Pipeline completed (OWASP scan skipped for now)."
        }
    }
}

// --------------------         original pipeline ------------------------------------
// pipeline {
//     agent any

//     environment {
//         SCANNER_HOME = tool 'sonar-scanner'
//         NVD_API_KEY = credentials('nvd-api-key')  // Jenkins secret text credential
//     }

//     tools {
//         maven 'maven3'
//         jdk 'jdk-17'
//     }

//     stages {
//         stage('git checkout') {
//             steps {
//                 git branch: 'main', url: 'https://github.com/Rajvardhan-128/Ekart.git'
//             }
//         }

//         stage('compile') {
//             steps {
//                 sh "mvn compile"
//             }
//         }

//         stage('unit tests') {
//             steps {
//                 sh "mvn test -DskipTests=true"
//             }
//         }

//         stage('SonarQube analysis') {
//             steps {
//                 withSonarQubeEnv('sonar-token') {
//                     sh "${env.SCANNER_HOME}/bin/sonar-scanner \
//                         -Dsonar.projectKey=EKART \
//                         -Dsonar.projectName=EKART \
//                         -Dsonar.java.binaries=target/classes"
//                 }
//             }
//         }

//         stage('OWASP Dependency Check') {
//             steps {
//                   withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
//                     dependencyCheck additionalArguments: "--nvdApiKey=$NVD_API_KEY",
//                                     odcInstallation: 'DC'
//              }
//         }
//         }

//         stage('Build') {
//             steps {
//                 sh "mvn package -DskipTests=true"
//             }
//         }

//         stage('deploy to Nexus') {
//             steps {
//                 withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk-17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
//                     sh "mvn deploy -DskipTests=true"
//                 }
//             }
//         }
        

//         stage('build and Tag docker image') {
//             steps {
//                 script {
//                         sh "docker build -t rock284/ekart:latest -f docker/Dockerfile ."
//                     }
//             }
//         }

//         stage('Push image to Hub'){
//             steps{
//                 script{
//                    withCredentials([string(credentialsId: 'dockerhub-pwd', variable: 'dockerhubpwd')]) {
//                    sh 'docker login -u rock284 -p ${dockerhubpwd}'}
//                    sh 'docker push rock284/ekart:latest'
//                 }
//             }
//         }
//         stage('EKS and Kubectl configuration'){
//             steps{
//                 script{
//                     sh 'aws eks update-kubeconfig --region ap-south-1 --name project-cluster'
//                 }
//             }
//         }
//         stage('Deploy to k8s'){
//             steps{
//                 script{
//                     sh 'kubectl apply -f deploymentservice.yml'
//                 }
//             }
//         }
//     }

// }
