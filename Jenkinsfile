pipeline {
  agent any
  stages {
    stage('Fetch Code') {
                steps {
                    git branch: 'dev', url: 'git clone https://kubernlinks@bitbucket.org/kubernlinks/pokt.git'
                }
            }
    stage('dev status check') {
      agent { docker { image 'liquibase/liquibase:4.17' } }
      steps {
        sh 'liquibase status --url="jdbc:postgresql://database-1.cpuc6bgspxr2.eu-central-1.rds.amazonaws.com:5432/postgres" --changeLogFile=changelog_version.xml --username=postgres --password=fellaini'
      }
    }
    stage('deploy') {
       steps {
        git branch: 'dev', url: 'git clone https://kubernlinks@bitbucket.org/kubernlinks/pokt.git'
        sh 'mvn clean package -Dmaven.test.skip=true'
        sh 'mvn liquibase:update'
        sh 'mvn liquibase:status -PDEPLOY'
        }
    }
    stage('Approval') {
            // no agent, so executors are not used up when waiting for approvals
             when { changeset "vm-management/create-vm/**"}
            agent none
            steps {
                script {
                    mail from: "$VM_EMAIL_FROM", to: "$VM_SUPPORT_EMAIL", subject: "APPROVAL REQUIRED FOR $JOB_NAME" , body: """Build $BUILD_NUMBER required an approval. Go to $BUILD_URL for more info."""
                    def deploymentDelay = input id: 'Deploy', message: 'Deploy to production?', parameters: [choice(choices: ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24'], description: 'Hours to delay deployment?', name: 'deploymentDelay')]
                    sleep time: deploymentDelay.toInteger(), unit: 'HOURS'
                }
            }
        }
    stage('Get Approval')  {
        options {
            timeout(time:  1, unit: 'MINUTES')
        }
        steps {
            input "please approve to  proceed to QA"
        }
     }

    stage('QA status check') {
        agent { docker { image 'liquibase/liquibase:4.17' } }
        steps {
           git branch: 'QA', url: 'git clone https://kubernlinks@bitbucket.org/kubernlinks/pokt.git'
           sh 'liquibase status --url="jdbc:postgresql://database-2.cpuc6bgspxr2.eu-central-1.rds.amazonaws.com:5432/postgres" --changeLogFile=changelog_version.xml --username=postgres --password=fellaini'
       }
    }
    stage('QA deploy') {
      steps {
        sh 'mvn clean package'
        sh 'mvn liquibase:update --url= --changeLogFile=$env:changeLogFile --username=postgres --password=fellaini'
        sh 'mvn liquibase:status -PQA'
      }
    }
    stage('Get Approval')  {
       options {
          timeout(time:  1, unit: 'days')
       }
       steps {
           input "please approve to  proceed to QA"
      }
    }
    stage('PROD status check') {
            agent { docker { image 'liquibase/liquibase:4.17' } }
            steps {
               git branch: 'PROD', url: 'git clone https://kubernlinks@bitbucket.org/kubernlinks/pokt.git'
               sh 'liquibase status --url="jdbc:posttgresql://prod.cpuc6bgspxr2.eu-central-1.rds.amazonaws.com:5432/postgres" --changeLogFile=changelog_version.xml --username=postgres --password=fellaini'
           }
        }
    stage('PROD deploy') {
          steps {
            sh 'mvn clean package'
            sh 'mvn liquibase:update --url=$env:QA_URL --changeLogFile=$env:changeLogFile --username=postgres --password=fellaini'
            sh 'mvn liquibase:status'
            sh 'mvn Spring-boot:run'
          }
    }
    stage('Update') {
      steps {
        sh 'liquibase update --url="$QA_URL" --changeLogFile=$changeLogFile --username=postgres --password=fellaini'
      }
    }
  }
  post {
    always {
      cleanWs()
    }
  }
}
