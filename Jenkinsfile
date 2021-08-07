pipeline{
    agent any
    
    environment{
        scannerHome = tool name: 'sonar_scanner_dotnet'
        registry = '3153712/gouravkansal_webapp'
        properties = null
        docker_port = '7300'
        username = 'gouravkansal'
        project_id = 'nagp-assignment-321709'
        cluster_name = 'dotnet-web-app'
        location = 'us-central1-c'
        credentials_id = 'KubernetesCredentials'
        }
    
    stages{
     stage('Checkout') {
       steps {
     git url: 'https://github.com/kansalgourav67/app_gouravkansal.git', branch: 'develop'
       }
     }
     
     stage('Restore packages'){
      steps{
      bat "dotnet restore WebApp\\WebApp.csproj"
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

    stage('Release artifacts'){
    steps{
        echo 'Release artifacts'
        }
        }
    
     
     stage('Docker Image'){
         steps{
             echo "Build docker image"
             bat 'dotnet publish -c Release'
             bat "docker build -t i-gouravkansal-develop:${BUILD_NUMBER} --no-cache -f Dockerfile ."
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
             bat "docker tag i-gouravkansal-develop:${BUILD_NUMBER} ${registry}:${BUILD_NUMBER}"
             bat "docker tag i-gouravkansal-develop:${BUILD_NUMBER} ${registry}:latest"
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
            bat "docker run --name c-gouravkansal-develop -d -p ${docker_port}:80 i-gouravkansal-develop:${BUILD_NUMBER}"
        }
    }
    
    stage('Kubernetes Deployment'){
        steps{
            step([$class: 'KubernetesEngineBuilder', projectId: env.project_id, clusterName: env.cluster_name, location: env.location, manifestPattern: 'deployment.yml', credentialsId: env.credentials_id, verifyDeployments: true]);
        }
    }
    
    }
}