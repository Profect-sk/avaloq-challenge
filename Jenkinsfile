
pipeline {
    agent any
    tools{
        maven 'Maven 3.6.3'
        jdk 'openjdk-11'
    }
    environment {
        PROJECT_NAME="avaloq-challenge"
    }
    stages {
	stage ('Build'){
	    steps {
		 githubNotify description: 'In Progress',  status: 'PENDING', context: 'Build'
	    	 sh 'mvn clean'
		 sh 'mvn compile'
	    }
	     post {
		failure {
                    githubNotify description: 'FAILED, check jenkins console for details',  status: 'FAILURE', context: 'Build'
                }
                success {
                    githubNotify description: 'OK',  status: 'SUCCESS', context: 'Build'
                }
            }
	}
        stage ('Unit Testing') {
            steps {
                githubNotify description: 'In Progress',  status: 'PENDING', context: 'Unit Testing'
                sh 'mvn -Dtest="**.unit.**" test'
            }
            post {
		always {
			junit 'target/surefire-reports/**/*.xml'
		}
		failure {
                    githubNotify description: 'FAILED, check jenkins console for details',  status: 'FAILURE', context: 'Unit Testing'
                }
                success {
                    githubNotify description: 'OK',  status: 'SUCCESS', context: 'Unit Testing'
                }
            }

        }
        stage ('Integration Tests') {
            when {
            anyOf { branch 'develop'; branch 'master'; branch 'main'; branch 'staging' }
            }
            steps {
	    	githubNotify description: 'In Progress',  status: 'PENDING', context: 'Integration Testing'
		sh 'mvn -Dtest="**.integration.**" test'
            }
	    post {
		always {
			junit 'target/surefire-reports/**/*.xml'
		}
		failure {
                    githubNotify description: 'FAILED, check jenkins console for details',  status: 'FAILURE', context: 'Integration Testing'
                }
                success {
                    githubNotify description: 'OK',  status: 'SUCCESS', context: 'Integration Testing'
                }
            }
        }
        stage('SonarQube analysis') {
            steps {
                githubNotify description: 'Performing Analysis',  status: 'PENDING', context: 'SonarQube Analysis'
                script {
                  scannerHome = tool 'sonar'
                  GIT_BRANCH = GIT_BRANCH.replaceAll('/','-')
                  print GIT_BRANCH
		  REPO_NAME = scm.getUserRemoteConfigs()[0].getUrl().tokenize('/').last().split("\\.")[0];
		  print REPO_NAME
                }
                withSonarQubeEnv('sonar') {
                  sh "${scannerHome}/bin/sonar-scanner " +
                      "-Dsonar.projectKey=${REPO_NAME}-${GIT_BRANCH} "+
                      "-Dsonar.projectName=${REPO_NAME}-${GIT_BRANCH} "+
                      "-Dsonar.sourceEncoding=UTF-8 "+
                      "-Dsonar.sources=src/main "+
                      "-Dsonar.java.binaries=target/classes,target/test-classes "+
                      "-Dsonar.tests=src/test "+
                      "-Dsonar.dynamicAnalysis=reuseReports "+
                      "-Dsonar.junit.reportsPath=target/sunfire-reports "+
                      "-Dsonar.java.coveragePlugin=jacoco "+
                      "-Dsonar.jacoco.reportPath=target "
                }
                timeout(time: 2, unit: 'MINUTES') {
          			script {
            			def qualityGate = waitForQualityGate()
            			if (qualityGate.status != 'OK') {
              				error "Pipeline aborted due to a quality gate failure: ${qualityGate.status}"
            			}
          			}
      			}
      			githubNotify description: 'OK',  status: 'SUCCESS', context: 'SonarQube Analysis'
            }
            post {
                failure {
                    githubNotify description: 'FAILED, check SonarQube for details',  status: 'FAILURE', context: 'SonarQube Analysis'
                }
                success {
                    githubNotify description: 'OK',  status: 'SUCCESS', context: 'SonarQube Analysis'
                }
            }
        }
    	stage ('Deploy to Staging/Test Environment') {
	        when {
		        branch 'staging'
	        }
	        steps {
		        echo 'run only when branch staging'
	        }
	    }
    	stage ('Deploy to Production Environment') {
            when {
		    anyOf{ branch 'master'; branch 'main'}
            }
            steps {
                echo 'run only when branch master'
            }
	    }
    }

}
