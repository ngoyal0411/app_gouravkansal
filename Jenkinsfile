pipeline{
    agent any
    
    environment{
        scannerHome = tool name: 'sonar_scanner_dotnet'
        registry = '3153712/gouravkansal_webapp'
        properties = null
        docker_port = '7200'
        username = 'gouravkansal'
        project_id = 'nagp-assignment-321709'
        cluster_name = 'dotnet-web-app'
        location = 'us-central1-c'
        credentials_id = 'KubernetesCredentials'
        }
    
    stages{
     stage('Checkout') {
       steps {
     git url: 'https://github.com/kansalgourav67/app_gouravkansal.git', branch: 'master'
       }
     }
     
     stage('Restore packages'){
      steps{
      bat "dotnet restore WebApp\\WebApp.csproj"
      }
     }
     
     stage('Start SonarQube Analysis'){
         steps{
             echo "SonarQube analysis started"
             withSonarQubeEnv("Test_Sonar"){
                 bat "${scannerHome}\\SonarScanner.MSBuild.exe begin /k:sonar-gouravkansal /n:sonar-gouravkansal /v:1.0"
             }
         }
     }
     
     stage('Clean Previous build'){
    steps{
        bat "dotnet clean WebApp\\WebApp.csproj"
    }
     }
     
    stage('Build the Code'){
   steps{
      bat "dotnet build WebApp\\WebApp.csproj"
   }
    }
    
    stage('Stop SonarQube Analysis'){
         steps{
             echo "SonarQube analysis stopped"
             withSonarQubeEnv("Test_Sonar"){
                 bat "${scannerHome}\\SonarScanner.MSBuild.exe end"
             }
         }
     }
     
     stage('Docker Image'){
         steps{
             echo "Build docker image"
             bat 'dotnet publish -c Release'
             bat "docker build -t i-gouravkansal-master:${BUILD_NUMBER} --no-cache -f Dockerfile ."
         }
     }
     
     stage('Containers'){
         parallel{
             stage('Precontainer Check'){
         steps{
             script {
                    if (env.BRANCH_NAME == 'develop') {
                        echo "remove container if running"
                        bat "docker rm --force /WebApp"
                    }
                }
         }
     }
     
     stage('Push Image to Docker Hub'){
         steps{
             echo 'Push image to docker hub'
             bat "docker tag i-gouravkansal-master:${BUILD_NUMBER} ${registry}:${BUILD_NUMBER}"
             bat "docker tag i-gouravkansal-master:${BUILD_NUMBER} ${registry}:latest"
             withDockerRegistry([credentialsId: 'DockerHub', url: ""]){
                 bat "docker push ${registry}:${BUILD_NUMBER}"
                 bat "docker push ${registry}:latest"
                }
            }
         }
      }
    }

    
    stage('Docker Deployment'){
        steps{
            echo 'Deploy to docker'
            bat "docker run --name c-gouravkansal-master -d -p ${docker_port}:80 i-gouravkansal-master:${BUILD_NUMBER}"
        }
    }
    
    stage('Kubernetes Deployment'){
        steps{
            bat "kubectl apply -f deployment.yaml"
        }
    }
    
    }
}