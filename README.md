# SQ_Docker_Webhook_CI-CDPipeline

1.	INSTALL JAVA for sonarqube(sudo apt install openjdk-17-jre)  and jenkins
2.	Install Jenkins
   
    a.	Create a new freestyle public project
  	
    b.	Go to manage Jenkins -- Configure -- Scm settings – git(provide repo link), branches to build – */master , build triggers – enable github hook trigger for gitscm polling and poll SCM--> apply & save
  	
    c.	Go to github repo -- settings -- webhooks -- add webhook -- path – http://192.168.1.104:8080/github-webhook/
  	
    d.	Build
  	
    e.	Make changes to github repo and check if build is triggered automatically.

4.	Download and Install sonarQube for windows 10 https://www.sonarsource.com/products/sonarqube/
   
    a.	Download the community edition and unzip the folder
  	
    b.	Go to bin --- windows –X86-64 --- StartSonar.bat
  	
    c.	 Go to conf --- sonar.properties and enable sonar.web.port 9000 under web servers
  	
    d.	go to browswer and check whether sonarqube is running on port 9000

            http://192.168.80.128:9000
            Login using default username and password - admin
            Reset credentials – admin, ****
            
    e.	Select Manually –> project display name and key – sonar_docker_webhook(both), branch- main --> set up
  	
    f.	Choose the baseline for newcode for this project – use the global setting – create project
  	
    g.	Analysis method – with jenkins
  	
    h.	Devops platform—github
  	
    i.	Analyze your project with Jenkins

          Steps 1 and 2 – continue
          Step 3 – other (as we are using html)
          Step 4 –create a jenkinsfile – select others for html,css,js
    
    j.	In step 4, now we need to do 2 things-

        Copy-paste the project key and also copy-paste the pipeline code in some text editor of your choice

    k.	Now go to admin user in top right corner --- select myaccount --- go to security and then generate a token with the following steps-

        Name - Sonarqube-token
        Type – Global analysis token
        Expiry – 30 days
        And Generate 
        
        sqa_6508c****

    l.	Go to Jenkins , install two plugins ---  ‘sonarqube scanner’ and ‘ssh2easy’
    
    m.	Manage Jenkins -- Tools -- sonarscanner installations -- Add sonarqube scanner -- Name- SonarScanner and install automatically
  	
    n.	Manage Jenkins -- System configuration -- system -- SonarQube Servers -- Add sonarqube -
    
        i.	Name – SonarServer1
        ii.	URL - http://192.168.1.104:9000/
        iii.	Server authentication key --- add jenkins –
              •	Kind – secret text
              •	Secret - sqa_6508c9abc572ba6009a572bb458b43a556907548
              •	ID  - sonar-token
              •	Add
              •	Save

    o.	Download and install SonarScanner from [https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/]for windows
  	
        i.	After downloading, extract the contents of the ZIP archive to a directory of your choice. This directory will contain the sonar-scanner executable.
        ii.	Go to /conf/sonar-scanner.properties and enable the default sonarqube server port
  	
    p.	Configure  Github Project --- copy-paste the repo url

    q.	Pipeline Script[for git and sonarqube]

                pipeline {
                    agent any
                
                    stages {
                        stage('Git Checkout') {
                            steps {
                                script {
                                    checkout scmGit(
                                        branches: [[name: '*/master']], 
                                        extensions: [], 
                                        userRemoteConfigs: [[url: 'https://github.com/namratasgit/SQ_Docker_Webhook_CI-CDPipeline.git']]
                                    )
                                }
                            }
                        }
                        
                        stage('SonarQube analysis') {
                            steps {
                                script {
                                    def scannerHome = 'C:/Users/user/Desktop/Namrata_Das_PU/Fall_AY_2023-24/DevOps/installers/sonar-scanner-5.0.1.3006-windows'
                                    def projectKey = 'sonar_docker_webhook' // Replace with your project key
                                    
                                    withSonarQubeEnv('SonarServer1') {
                                        bat "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=${projectKey}"
                                    }
                                }
                            }
                        }   
                    }
                }
                        

6.	Download and install docker
   
pipeline {
    agent any

    stages {
        stage('Git Checkout') {
            steps {
                script {
                    checkout scmGit(
                        branches: [[name: '*/master']], 
                        extensions: [], 
                        userRemoteConfigs: [[url: 'https://github.com/namratasgit/SQ_Docker_Webhook_CI-CDPipeline.git']]
                    )
                }
            }
        }
        
        stage('SonarQube analysis') {
            steps {
                script {
                    def scannerHome = 'C:/Users/user/Desktop/Namrata_Das_PU/Fall_AY_2023-24/DevOps/installers/sonar-scanner-5.0.1.3006-windows'
                    def projectKey = 'sonar_docker_webhook' // Replace with your project key
                    
                    withSonarQubeEnv('SonarServer1') {
                        bat "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=${projectKey}"
                    }
                }
            }
        }
        stage("Build Docker Image") {
            steps 
            {
                script {
                    // Navigate to the directory containing the Dockerfile
                    dir('C:/Users/user/Desktop/Namrata_Das_PU/Fall_AY_2023-24/DevOps/MyGit/calculator') 
                    {
                        // Build the Docker image
                        bat 'docker build -t calculator_app_image .'
                    }
                }
            }
        }
        stage("Push image to hub") {
            steps {
                script {
                    withCredentials([string(credentialsId: 'secret', variable: 'dockerhubpwd')]) {
                        bat "docker login -u namratasdocker -p ${dockerhubpwd}"
                        bat "docker tag 034469d3cc45 namratasdocker/sonar_webhook_docker_pipeline:latest"

                        bat "docker push namratasdocker/sonar_webhook_docker_pipeline"
                    }
                }
            }
            
        }


    }
}

5. Build
