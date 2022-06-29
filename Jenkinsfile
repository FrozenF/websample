pipeline {
    agent any

    environment {
      HARBOR_URL = 'your-harbor-url'
      HARBOR_USER = credentials('btech-harbor-user') 
      HARBOR_PASS = credentials('btech-harbor-pass')
    }

    stages {
      stage('Build') {
        steps {
            sh "dotnet restore"
            sh "dotnet publish -c Release -o out"
        }
      }
      stage('Unit Testing') {
        steps {
            sh "dotnet test"
        }
      }
      stage('SCA') {
        steps {
          script {
            env.PATH = "$PATH:$HOME/.dotnet/tools"
          }
          sh 'dotnet CycloneDX . -o .'
          dependencyTrackPublisher artifact: 'bom.xml', autoCreateProjects: true, dependencyTrackApiKey: 'dt-api-key', dependencyTrackFrontendUrl: 'http://dt.btech.id', dependencyTrackUrl: 'http://dt.btech.id:8081', overrideGlobals: true, projectName: 'DotNetWebSample', projectVersion:'1', synchronous: true
        }
      }
      stage('SAST') {
        steps {
          script {
            scannerHome = tool 'SonarScannerMSBuild5.7';
          }
          withSonarQubeEnv('SonarQubeServer') {
            sh "dotnet ${scannerHome}/SonarScanner.MSBuild.dll begin /k:\"dot-net-web\""
            sh "dotnet build"
            sh "dotnet ${scannerHome}/SonarScanner.MSBuild.dll end"
          }
        }
      }
      stage("SAST - Result") {
          steps {
              waitForQualityGate abortPipeline: true
          }
      }
      stage('Build Push'){
        steps {
          sh 'docker login $HARBOR_URL -u $HARBOR_USER -p $HARBOR_PASS'
          sh 'docker build . -t $HARBOR_URL/dotnet/webproject:$BRANCH_NAME -t $HARBOR_URL/dotnet/webproject:latest'
          sh 'docker push $HARBOR_URL/dotnet/webproject:$BRANCH_NAME'
          sh 'docker push $HARBOR_URL/dotnet/webproject:latest'
        }
      }
    }
 }
