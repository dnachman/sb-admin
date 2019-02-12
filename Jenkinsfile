def config = [	dockerRepository: 'logicalenigma/',
                dockerRegistryCredentials: 'docker-hub']

def pomArtifactId
def pomVersion
pipeline {
	agent any
	stages{
		stage('Build') {
			steps {
				// Run the maven build
      	sh "./mvnw -Dmaven.test.failure.ignore clean package"
				script {
					pomVersion = readMavenPom().getVersion()
					pomArtifactId = readMavenPom().getArtifactId()
				}
			}
   	}
		stage('Package/Push image') {
			parallel {
				stage ('amd64') {
					steps {
						script {
								// build docker image
							def dockerImage = docker.build(config.dockerRepository + "${pomArtifactId}:${pomVersion}")
							docker.withRegistry('', config.dockerRegistryCredentials) {
								dockerImage.push()
								dockerImage.push('latest')
							}
			    	}
					}
				}
				stage ('rpi') {
					steps {
						script {
								// build docker image
							def dockerImageRpi = docker.build(config.dockerRepository + "${pomArtifactId}" + ":rpi-${pomVersion}", '-f Dockerfile.rpi .')
							docker.withRegistry('', config.dockerRegistryCredentials) {
								dockerImageRpi.push()
								dockerImageRpi.push('rpi')
							}
			    	}
					}
				}
			}
			
		}
		stage('Remove Unused docker image') {
            steps{
                sh "docker rmi ${config.dockerRepository}${pomArtifactId}:${pomVersion}"
                sh "docker rmi ${config.dockerRepository}${pomArtifactId}:rpi-${pomVersion}"
            }
        }
		stage('Results') {
			steps {
				junit '**/target/surefire-reports/TEST-*.xml'
			}
		}
	}
}