pipeline {
    agent { label 'master'}
    stages {
        stage('Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/hmtrung/WebDemo.git']]])
            }
        }
        stage('Build image') {
            steps {
                sh label: '', script: '''
                    docker build --build-arg http_proxy=http://10.10.10.10:8080 --build-arg https_proxy=http://10.10.10.10:8080 -t my-python-app:${BUILD_NUMBER} .
                    docker tag my-python-app:${BUILD_NUMBER} my-python-app:latest
                    '''
            }
        }
        stage('Clean up') {
            parallel {
                stage('Clean up existed app') {
                    steps {
                        sh label: '', returnStatus: true, script: 'docker stop my-running-app'
                    }
                }
                stage('Clean up existed Selenium server') {
                    steps {
                        sh label: '', returnStatus: true, script: 'docker stop myselenium'
                        sh label: '', returnStatus: true, script: 'docker network rm mynet'
                    }
                }            
            }            
        }
        stage('Run app') {
            steps {
                sh label: '', script: 'docker network create mynet'
                sh label: '', script: 'docker run -d --rm -p 7272:7272 --net mynet --name my-running-app my-python-app:${BUILD_NUMBER}'
            }
        }
        
        stage('Setup Selenium Server') {
            steps {
                sh label: '', returnStatus: true, script: 'docker stop myselenium'
                sh label: '', script: 'docker run -d --rm -p 4444:4444 -p 5900:5900 --net mynet -v /dev/shm:/dev/shm -v ${PWD}:/opt/WebDemo --name myselenium hmtrung/selenium-standalone-chrome-debug:3.141.59-radium'
                sh label: '', script: 'docker exec -i myselenium robot --outputdir /opt/WebDemo/results --variable BROWSER:chrome --variable SERVER:my-running-app:7272 /opt/WebDemo/login_tests'
            }
        }
        stage('Publish Robot Framework Results') {
            steps {
                step([$class: 'RobotPublisher',
                    outputPath: 'results',
                    passThreshold: 100,
                    unstableThreshold: 0,
                    otherFiles: ""])
            }
        }
        stage('Clean up Grid') {
            steps {
                sh label:'', script: 'docker stop myselenium'
            }
        }
        stage('Stop app') {
            steps {
                sh label: '', script: 'docker stop my-running-app'
            }
        }
        stage('Push image to registry') {
            steps {
                sh label: '', script: 'echo tests'
            }
        }
    }
}
