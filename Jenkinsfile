pipeline {
  agent any
  environment {
    devdb_cr=credentials('dev_db')
    qadb_cr=credentials('qa_db')
    proddb_cr=credentials('prod_db')
    dev_db="jdbc:postgresql://database-1.cpuc6bgspxr2.eu-central-1.rds.amazonaws.com:5432/postgres"
    qa_db="jdbc:postgresql://database-2.cpuc6bgspxr2.eu-central-1.rds.amazonaws.com:5432/postgres"
    prod_db="jdbc:postgresql://prod.cpuc6bgspxr2.eu-central-1.rds.amazonaws.com:5432/postgres"
  }
  stages {
    stage('Fetch Code') {
                steps {
                    git branch: 'dev', url: 'https://github.com/kubernlinks/pokt.git'                }
            }
    stage('dev status check & update') {
      agent { docker { image 'liquibase/liquibase:4.17' } }
      steps {
        git branch: 'dev', url: 'https://github.com/kubernlinks/pokt.git' 
        sh 'liquibase status --url=$dev_db --changeLogFile=dev_wrapper.xml --username=${devdb_cr_USR} --password=${devdb_cr_PSW}'
        //sh 'liquibase update --url=$dev_db --changeLogFile=dev_wrapper.xml --username=$devdb_cr_USR --password=$devdb_cr_PSW'   
      }
    }
    stage('Approval to QA')  {
        options {
            timeout(time:  1, unit: 'DAYS')
        }
        steps {
            input "please approve to  proceed to QA"
        }
     }

    stage('QA status check') {
        agent { docker { image 'liquibase/liquibase:4.17' } }
        steps {
           git url: 'https://github.com/kubernlinks/pokt.git', branch: 'QA'
           sh 'liquibase status --url=$qa_db --changeLogFile=QA_wrapper.xml --username=$qadb_cr_USR --password=$qadb_cr_PSW'
        }
     }
   
    stage('QA update') {
        steps {
           git url: 'https://github.com/kubernlinks/pokt.git', branch: 'QA'
           sh ' mvn liquibase:status -PQA'
           }
       }
    
    stage('Approval to Prod')  {
       options {
          timeout(time:  1, unit: 'DAYS')
       }
       steps {
           input "please approve to  proceed to PROD"
      }
    }
    stage('PROD status check & update') {
            agent { docker { image 'liquibase/liquibase:4.17' } }
            steps {
               git branch: 'PROD', url: 'https://github.com/kubernlinks/pokt.git'
               sh 'liquibase status --url=$prod_db --changeLogFile=prod_wrapper.xml --username=$proddb_cr_USR --password=$proddb_cr_PSW'
               sh 'liquibase update --url=$prod_db --changeLogFile=prod_wrapper.xml --username=$proddb_cr_USR --password=$proddb_cr_PSW'
            }
        }
  }
  post {
    always {
      cleanWs()
    }
  }
}
