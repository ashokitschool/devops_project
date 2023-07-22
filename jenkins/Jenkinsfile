def registry = ' https://ashokit.jfrog.io'
def imageName = 'ashokit.jfrog.io/ashokit-docker-local/insta'
def version   = '2.1.4'

pipeline {
    agent {
        node {
            label 'maven'
        }
    }
    
    environment{
        PATH = "/opt/apache-maven-3.9.3/bin:$PATH"
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/ashokitschool/ashokit_insta_app.git'
            }
        }
        
        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        
        stage(" Docker Build ") {
            steps {
                script {
                    echo '<--------------- Docker Build Started --------------->'
                    app = docker.build(imageName+":"+version)
                    echo '<--------------- Docker Build Ends --------------->'
                }   
            }
        }
        
        stage (" Docker Publish "){
            steps {
                script {
                    echo '<--------------- Docker Publish Started --------------->'  
                    docker.withRegistry(registry, 'jfrog-cred'){
                        app.push()
                    }    
                    echo '<--------------- Docker Publish Ended --------------->'  
                }
            }
        }
        
        stage (" Deployment "){
            steps {
                script {
                    echo '<--------------- Deployment Started --------------->'  
                    sh 'sh deploy.sh'
                    echo '<--------------- Deployment Ended --------------->'  
                }
            }
        }
        
    }
}
