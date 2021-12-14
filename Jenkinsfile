pipeline {
  agent any

  tools {
    jdk 'jdk11'
    maven '3.6.2' //đổi tên
  }

  stages {
    stage('Build && Test') {
      steps {
        withMaven(maven : '3.6.2') {
          sh "mvn package"
        }
      }
    }

//     stage ('OWASP Dependency-Check Vulnerabilities') {
//       steps {
//         withMaven(maven : 'mvn-3.6.3') {
//           sh 'mvn dependency-check:check'
//         }

//         dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
//       }
//     }
    stage ('OWASP Dependency-Check Vulnerabilities') {
      steps {
        dependencyCheck additionalArguments: ''' 
          -o "./" 
          -s "./"
          -f "ALL" 
          --prettyPrint''', odcInstallation: 'dependency-check'

        dependencyCheckPublisher pattern: 'dependency-check-report.xml'
      }
    }       
    stage('SonarQube analysis') {
      environment {
                scannerHome = tool 'SonarQubeScanner'
      }
      steps {
//         withSonarQubeEnv(credentialsId: 'sonarqube-secret', installationName: 'sonarqube-server') {
        withSonarQubeEnv('sonarqube') {
          withMaven(maven : '3.6.2') {
            sh 'mvn sonar:sonar -Dsonar.dependencyCheck.jsonReportPath=target/dependency-check-report.json -Dsonar.dependencyCheck.xmlReportPath=target/dependency-check-report.xml -Dsonar.dependencyCheck.htmlReportPath=target/dependency-check-report.html'
          }
        }
      }
    }
    stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    
    stage('Create container') {
      steps {
          withMaven(maven : '3.6.2') {
            sh "mvn compile jib:dockerBuild"
        }
      } 
    }

    stage("docker_scan"){
            steps {
                // docker run --network milestone -d --name db arminc/clair-db
                // sleep 15
                // docker run --network milestone -p 6060:6060 --link db:postgres -d --name clair arminc/clair-local-scan
                // sleep 1
                // wget -qO clair-scanner https://github.com/arminc/clair-scanner/releases/download/v8/clair-scanner_linux_amd64 && chmod +x clair-scanner
                sh '''
                    DOCKER_GATEWAY=$(docker network inspect bridge --format "{{range .IPAM.Config}}{{.Gateway}}{{end}}")
                    wget -qO clair-scanner https://github.com/arminc/clair-scanner/releases/download/v8/clair-scanner_linux_amd64 && chmod +x clair-scanner
                    ./clair-scanner --ip="$DOCKER_GATEWAY" milestone/sprint:latest || exit 0
                '''
            }
        }
    
    stage('Run container') {
          steps {
                sh "docker run --network milestone -p 8080:8080 --ip 172.18.0.100 -d --name milestone milestone/spring-boot-demo"
          } 
        }
    
//     stage('Create container') {
//       steps {
//         withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
//           withMaven(maven : '3.6.2') {
//             sh "mvn jib:build"
//           }
//         }
//       } 
//     }

//     stage('Anchore analyse') {
//       steps {
//         writeFile file: 'anchore_images', text: 'docker.io/maartensmeets/spring-boot-demo'
//         anchore name: 'anchore_images'
//       }
//     }

//     stage('Deploy to K8s') {
//       steps {
//         withKubeConfig([credentialsId: 'kubernetes-config']) {
//           sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"'
//           sh 'chmod u+x ./kubectl'
//           sh './kubectl apply -f k8s.yaml'
//         }
//       } 
//     }
  }
}
