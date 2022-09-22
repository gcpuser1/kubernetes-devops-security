pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }
      stage("Code Coverage and testing"){
        steps{
          sh "mvn test"
        }
        post{
          always{
            junit 'target/surefire-reports/**.xml'
            jacoco execPattern: 'target/**.exec'
          }
        }
      }
      stage("Mutataion Tests"){
        steps{
          sh "mvn test-compile org.pitest:pitest-maven:mutationCoverage"
        }
        post{
          always{
            pitmutation mutationStatsFile: 'target/pit-reports/**/mutations.xml'
          }
        }
      }
      stage("sonar scanner"){
        steps{
          withSonarQubeEnv('SonarQube'){
            sh "mvn clean verify sonar:sonar \
              -Dsonar.projectKey=devsec-ops"
          }
          //  timeout(time: 4, unit: 'MINUTES') {
          //           script{
          //             waitForQualityGate abortPipeline: true
          //           }
          //  }
        }
      } 


      stage("Docker Build and Push"){
        steps{
          withDockerRegistry([credentialsId: "docker", url:""]){
            sh "printenv"
            sh 'docker build -t kona1700/devsec-ops-""$GIT_COMMIT"" .'
            sh 'docker push kona1700/devsec-ops-""$GIT_COMMIT""'
          }
        }
      }
  }
}