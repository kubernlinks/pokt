pipeline {
  agent any
  environment {
    dev_db=credentials('dev_db')
    qa_db=credentials('qa_db')
    prod_db=credentials('prod_db')
  }
  stages {
    stage('Fetch Code') {
                steps {
                    git branch: 'dev', url: 'https://github.com/kubernlinks/pokt.git'                }
            }
    stage('dev status check & update') {
      agent { docker { image 'liquibase/liquibase:4.17' } }
      steps {
        sh 'liquibase status --url="jdbc:postgresql://database-1.cpuc6bgspxr2.eu-central-1.rds.amazonaws.com:5432/postgres" --changeLogFile=changelog_version.xml --username=${dev_db_USR} --password=${dev_db_PSW}'
        //sh 'liquibase update --url="jdbc:postgresql://database-1.cpuc6bgspxr2.eu-central-1.rds.amazonaws.com:5432/postgres" --changeLogFile=changelog_version.xml --username=postgres --password=fellaini'   
      }
    }
    stage('Approval. to. QA')  {
        options {
            timeout(time:  1, unit: 'DAYS')
        }
        steps {
            input "please approve to  proceed to QA"
        }
     }

    stage('QA status check & update') {
        agent { docker { image 'liquibase/liquibase:4.17' } }
        steps {
           checkout git url: 'https://github.com/kubernlinks/pokt.git', branch: 'QA'
           sh 'liquibase status --url="jdbc:postgresql://database-2.cpuc6bgspxr2.eu-central-1.rds.amazonaws.com:5432/postgres" --changeLogFile=changelog_version.xml --username=$qa_db_USR --password=$qa_db_PSW'
           sh 'liquibase update --url="jdbc:postgresql://database-2.cpuc6bgspxr2.eu-central-1.rds.amazonaws.com:5432/postgres" --changeLogFile=QA_wrapper.xml --username=$qa_db_USR --password=$qa_db_PSW'
       }
    }
    stage('Approval to Prod')  {
       options {
          timeout(time:  1, unit: 'DAYS')
       }
       steps {
           input "please approve to  proceed to QA"
      }
    }
    stage('PROD status check &. update') {
            agent { docker { image 'liquibase/liquibase:4.17' } }
            steps {
               git branch: 'PROD', url: 'https://github.com/kubernlinks/pokt.git'
               sh 'liquibase status --url="jdbc:postgresql://prod.cpuc6bgspxr2.eu-central-1.rds.amazonaws.com:5432/postgres" --changeLogFile=changelog_version.xml --username=postgres --password=fellaini'
               sh 'liquibase update --url="jdbc:postgresql://prod.cpuc6bgspxr2.eu-central-1.rds.amazonaws.com:5432/postgres" --changeLogFile=changelog_version.xml --username=postgres --password=fellaini'
            }
        }
  }
  post {
    always {
      cleanWs()
    }
  }
}
