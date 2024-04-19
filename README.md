# Jenkins

## Introduction
Jenkins is an open source continuous integration/continuous delivery and deployment (CI/CD) automation software DevOps tool written in 
the Java programming language. It is used to implement CI/CD workflows, called pipelines.
Pipelines automate testing and reporting on isolated changes in a larger code base in real time and facilitates the integration of 
disparate branches of the code into a main branch. They also rapidly detect defects in a code base, build the software, automate 
testing of their builds, prepare the code base for deployment (delivery), and ultimately deploy code to containers and virtual machines, 
as well as bare metal and cloud servers

## Jenkins Infrastracture
Jenkins infrastracture is very simple. There is a master server who controls pipelines and schedules builds and the agents (or minions)
who perform the build.
Said that, we can summary the entire Jenkins pipeline workflow works as follow:

1) In a Git repository (e.g. GitHub) a new commit triggers the pipeline
2) The Jenkins master, triggered, choose the agent based on configured labels to run the build
3) The selected agents launch the comands to build, test and deploy the application.

## Jenkins File
JenkinsFile is a file where we can describe our jenkins pipeline and it must be in our repo with our code.
The JenkinsFile can be of two types:

1) Scripted
Based on Groovy language, it give us the ability to write our Jenkins pipeline using advanced scripting capabilities and high flexibility.
The Scripted syntax is:
	node{
		// our Jenkins pipeline description
	}
It harder to use at the beginning than the Declaraive way.


2) Declarative
If you don't know Groovy you can use the declarative way, that is easier to use at the beginning but is less powerful.
The Declarative syntax is:
	pipeline{
		// our Jenkins pipeline description
	}

For the sake of simplicity, we'll use the Declarative syntax for the project, so let's explain the main part of a Declarative JenkinsFile.

### Declarative JenkinsFile structure
The simpler structure of a JenkinsFile using the Declarative syntax is:
	pipeline{
		
		agent any
		
		stages {
		
			stage("build") {
			
				steps {
					echo 'building the application'
				}
			
			}
			
			stage("test") {
			
				steps {
					echo 'testing the application'
				}
			
			}
		
			stage("deploy") {
			
				steps {
					echo 'deploying the application'
				}
			
			}		
		
		
		}
		
	}

Let's explain the various terms we find in the JenkinsFile structure.

* pipeline is the word with wich the jenkinsfile must start. It represent the beginning of the Jenkins pipeline description.
* agents this describe where the build must be executed. Any represents that the pipeline we'll be executed on any available Jenkins agent
* stages represent the real description of what the Jenkins pipeline will do. Usually we have a build, a test and a deploy stages.
* steps is the place where we can describe what a stage does. In the previous example it just print something.


### Other fetaures used in JenkinsFile
But we have much more features on our Jenkinsfile!
In the following line we will see a bunch of main features that are really useful.

**1) Post Build** <br />
As we can see in the following Jenkins structure, we can add a step after the stages one where we can define what Jenkins have to do after 
the build. We can also specify what script execute based on the status of the pipeline building (failure or success).

	pipeline{
		
		agent any
		
		stages {
		
			stage("build") {
			
				steps {
					echo 'building the application'
				}
			
			}
			
			stage("test") {
			
				steps {
					echo 'testing the application'
				}
			
			}
		
			stage("deploy") {
			
				steps {
					echo 'deploying the application'
				}
			
			}		
		
		
		}
		post {
			
			success{
				// script executed only if the build ended with a success
			}
			failure{
				// script executed only if the build ended with a failure
			}	
			always {
				// script always executed
			}
			
		}
		
		
	}


**2) Conditional Stages** <br />
What if we want to execute a pipeline stage only if a certain condition is match?
We can use the when expression! The when statment is used before the step declaration in a stage declaration. We can see it as a simple
if condition. The step statment is executed if the condition in the when statment is matched.

	pipeline{
		
		agent any
		
		stages {
		
			stage("build") {
			
				steps {
					echo 'building the application'
				}
			
			}
			
			stage("test") {
				
				when {
					expression {
						BRANCH_NAME == 'dev' || BRANCH_NAME == 'master' 
					}	
				}
				
				steps {
					echo 'testing the application'
				}
			
			}
		
			stage("deploy") {
			
				steps {
					echo 'deploying the application'
				}
			
			}		
		
		
		}
		
		
	}
	
	
**3) Environment Variable** <br />
Jenkins provide us a lot of environment variable that we can use in our pipeline, for example the BRANCH_NAME that we used in the 
previous point. We can also define a custom one using the environment statment as we can see in the following syntax.
All the variable defined in the environment are vailable for all the stage defined in the stages.
To use the variable in the stage is using the ${variable name} syntax.
Be aware of the double quote in the steps statment to use the vailable!

	pipeline{
		
		agent any
		
		environment {
		
			NEW_VERSION = '1.0.0'
		
		}
		
		stages {
		
			stage("build") {
			
				steps {
					echo 'building the application'
					echo "building version ${NEW_VERSION}"  
				}
			
			}
			
			stage("test") {
				
				when {
					expression {
						BRANCH_NAME == 'dev' || BRANCH_NAME == 'master' 
					}	
				}
				
				steps {
					echo 'testing the application'
				}
			
			}
		
			stage("deploy") {
			
				steps {
					echo 'deploying the application'
				}
			
			}		
		
		
		}
		
		
	}
	
	
**4) Build Tools** <br />
The tools attribute let us to specify what build tools (maven, gradle and JDK) we gonna use in the pipeline. 
To define the tools to use in th pipeline stage is using the tools statment where we have to specify the name of the build tool.
After that we can use the build tool functionality in the stage step using the sh "build tool functionality".
Let's see in the structure...

NOTE!
The build tools available are maven, gradle and jdk. If you want to use other build tools you have to do it in other way.

NOTE!
Maven, gradle and JDK are preconfigured in Jenkins!


	pipeline{
		
		agent any
		
		tools {
		
			maven
		
		}
		
		stages {
		
			stage("build") {
			
				steps {
					echo 'building the application'
					sh "mvn install"
				}
			
			}
			
			stage("test") {
				
				steps {
					echo 'testing the application'
				}
			
			}
		
			stage("deploy") {
			
				steps {
					echo 'deploying the application'
				}
			
			}		
		
		
		}
		
		
	}
	
	
**5) Parameters** <br />
We can define parameters to use in our pipeline. We can do that using the parameters statment.


	pipeline{
		
		agent any
		
		parameters {
			string(name: 'VERSION', defaultValue:'', description: 'version to deploy on prod')
			choice(name: 'VERSION', choices: ['1.0.0', '1.1.0', '1.2.0'], description: '')
			booleanParam(name: 'executeTests', defaultValue: true, description: '')
		}
		
		stages {
		
			stage("build") {
			
				steps {
					echo 'building the application'
				}
			
			}
			
			stage("test") {
				
				when {
					expression {
						params.executeTests == true					
					}	
				}
				
				steps {
					echo 'testing the application'
				}
			
			}
		
			stage("deploy") {
			
				steps {
					echo 'deploying the application'
				}
			
			}		
		
		
		}
		
		
	}


**5) External Scripts** <br />
We can also use external Groovy script in our JenkinsFile. To do that we have to use the script statment.

	pipeline{
		
		agent any
	
		
		stages {
		
			stage("build") {
			
				steps {
					
					script {
						// load or write the Groovy script
					}
					
				}
			
			}
			
			stage("test") {
							
				steps {
					echo 'testing the application'
				}
			
			}
		
			stage("deploy") {
			
				steps {
					echo 'deploying the application'
				}
			
			}		
		
		
		}
		
		
	}


## The project...
The project follows these steps:

1) create and push on GitHub a new simple Quarkus app that expose just one GET endpoint
2) for every new commit the app will be dockerized and pushed on Docker Hub
3) the new version of the app will be pulled from Docker Hub and deployed on Minikube (Local Kubernetes)

All the previous steps will be automated using a Jenkins pipeline.

**Note!**
Pay attention if you are a Mac user, you must modify a file under Cellar folder to run docker inside jenkins pipeline.

**Note!**
Pay attention that you have to install a plugin that is deprecated in Jenkins to be able to deploy on Kubernetes. You can find the plugin in the project folder and you have to upload manually in Jenkins plugin using the advanced section.

**Note!**
Remember that to contact the Kubernetes service you need to open a tunnel against it using the command:
minikube service simple-jenkins-app-service --url