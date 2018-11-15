pipeline {
  agent any

  stages {
        stage("Compilation and Analysis") {
            parallel {
                stage('Compilation') {    
                    steps {
                            sh "./mvnw clean install -DskipTests"
                        }
                }

                    stage("Checkstyle") {
                        steps {
                            sh "./mvnw checkstyle:checkstyle"
                            
                            step([$class: 'CheckStylePublisher',
                            canRunOnFailed: true,
                            defaultEncoding: '',
                            healthy: '100',
                            pattern: '**/target/checkstyle-result.xml',
                            unHealthy: '90',
                            useStableBuildAsReference: true
                            ])
                        }
                    }
                }
            }
         
        stage("Tests and Deployment") {
            parallel {
                stage("Runing unit tests") {
                    try {
                        sh "./mvnw test -Punit"
                    } catch(err) {
                        step([$class: 'JUnitResultArchiver', testResults: 
                          '**/target/surefire-reports/TEST-*UnitTest.xml'])
                        throw err
                    }
                   step([$class: 'JUnitResultArchiver', testResults: 
                     '**/target/surefire-reports/TEST-*UnitTest.xml'])
                }
                stage("Runing integration tests") {
                    try {
                        sh "./mvnw test -Pintegration"
                    } catch(err) {
                        step([$class: 'JUnitResultArchiver', testResults: 
                          '**/target/surefire-reports/TEST-'
                            + '*IntegrationTest.xml'])
                        throw err
                    }
                    step([$class: 'JUnitResultArchiver', testResults: 
                      '**/target/surefire-reports/TEST-'
                        + '*IntegrationTest.xml'])
                }
            }
             
            stage("Staging") {
                sh "pid=\$(lsof -i:8989 -t); kill -TERM \$pid "
                  + "|| kill -KILL \$pid"
                withEnv(['JENKINS_NODE_COOKIE=dontkill']) {
                    sh 'nohup ./mvnw spring-boot:run -Dserver.port=8989 &'
                }   
            }
        }
    }
}

