pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: gcloud-agent
    image: docker.io/rulihongran/jenkins-gcloud-agent:v1
    command:
    - cat
    tty: true
"""
        }
    }

    environment {
        SONAR_SERVER = 'sonarqube-server'
        PROJECT_ID = 'gothic-tempest-490002-j3'
        REGION = 'us-central1'
        DATAPROC_CLUSTER = 'hadoop-cluster'
        GCS_SCRIPT_PATH = 'gs://mayavi-dataproc-scripts/hello_hadoop.py'
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'my-sonar-scanner'

                    withSonarQubeEnv("${SONAR_SERVER}") {

                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=mayavi \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=${SONAR_AUTH_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Run Hadoop Job') {
            steps {
                container('gcloud-agent') {

                    echo "Quality Gate passed. Running Dataproc job..."

                    sh """
                    gcloud dataproc jobs submit pyspark \
                        --cluster=${DATAPROC_CLUSTER} \
                        --region=${REGION} \
                        ${GCS_SCRIPT_PATH}
                    """
                }
            }
        }
    }
}