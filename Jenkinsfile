pipeline {
    agent any
    parameters {
        string(name: 'MYSQL_ROOT_PASSWORD', defaultValue: 'root', description: 'MySQL password')
    }
    stages {
        stage ("Initialize Jenkins Env") {
         steps {
            bat 'echo %JAVA_HOME%'
            bat '''
            echo "PATH = ${PATH}"
            echo "M2_HOME = ${M2_HOME}"
            '''
         }
        }
        stage('Download Code') {
            steps {
               echo 'checking out'
               checkout scm
            }
        }
        stage('Execute Tests'){
            steps {
                echo 'Testing'
                bat 'mvn test'
            }
        }
        stage('Build Application'){
            steps {
                echo 'Building...'
                bat 'mvn clean install -Dmaven.test.skip=true'
            }
        }
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image'
                bat 'docker build -t hendisantika/online-banking:1 .'
            }
        }
       stage('Create Database') {
            steps {
                echo 'Running Database Image'
            //    sh 'docker kill bankmysql 2> /dev/null'
            //    sh 'docker kill cloudbank 2> /dev/null'
            //    sh 'docker rm bankmysql 2> /dev/null'
            //    sh 'docker rm cloudbank 2> /dev/null'
                bat 'docker rm -f bankmysql || true'
                bat 'docker run --detach --name=bankmysql --env="MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}" -p 3305:3305 mysql'
                bat 'ping -n 20 127.0.0.1'
            //  sh 'docker exec -i bankmysql mysql -uroot -proot < sql_dump/onlinebanking.sql'
                bat 'docker exec -i bankmysql mysql -uroot -p${MYSQL_ROOT_PASSWORD} < sql_dump/onlinebanking.sql'
            }
        }
        stage('Deploy and Run') {
            steps {
                echo 'Running Application'
                bat 'docker stop cloudbank || true && docker rm cloudbank || true'
                bat 'docker run --detach --name=cloudbank -p 8888:8888 --link bankmysql:localhost -t hendisantika/online-banking:1'
            }
        }
    }
}
