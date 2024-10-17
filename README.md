
@Library("Share") _

pipeline{
    agent any

    environment{
        SONAR_HOME = tool "Sonar"
    }
    stages{
       
        stage("code"){
            steps{
              echo "clone the code"  
              git url:"https://github.com/namitrohin/connectx_api.git" , branch:"main"
              echo "code cloning"
            }
        }
        stage("Trivy: Filesystem scan"){
            steps{
                script{
                    trivy_scan()
                }
            }
        }
        

        stage("OWASP: Dependency check"){
            steps{
                script{
                    owasp_dependency()
                }
            }
        }
        
        stage("SonarQube: Code Analysis"){
            steps{
                script{
                    sonarQube_analysis("Sonar","connectx_api","connectx_api")
                }
            }
        }
        
        // stage("SonarQube: Code Quality Gates"){
        //     steps{
        //         script{
        //             sonarQube_code_quality()
        //         }
        //     }
        // }

        
        stage("build"){
           steps{
               echo "build code"
            //   sh "whoami"
              sh "docker build -t connectx_api:latest ."
              echo "docker build successfully"
           } 
        }
        stage("Push to dockerHub"){
            steps{
                withCredentials([usernamePassword(credentialsId:"dockerHubCred",passwordVariable:"dockerHubPass",usernameVariable:"dockerHubUser")]){
                sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                sh "docker tag connectx_api:latest ${env.dockerHubUser}/connectx_api:latest"
                sh "docker push ${env.dockerHubUser}/connectx_api:latest"
                echo 'image push ho gaya'
                }
            }
            
        }
        stage("deploy"){
          steps{
               sh "docker-compose up -d"
               echo "code Deploy"
            }  
        }
    }
}
