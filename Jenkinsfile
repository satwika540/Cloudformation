pipeline {
  agent any 
  environment {
      AWS_REGION = 'us-east-1'
  }
  parameters {
    choice(name: 'Environment', choices: ['dev', 'tst', 'prod'], description:'Select environment')
    string(name: 'GitCommitID', defaultValue: '', description: 'Git commit id')
    string(name: 'AppVersion', defaultValue:'1.0.0', description: 'App Version')
    string(name: 'VpcCIDR', defaultValue: '10.0.0.0/16', description: 'VPC CIDR Block')
    string(name: 'Subnet1CIDR', defaultValue: '10.0.1.0/24', description: 'subnet1 cidr')
    string(name: 'Subnet2CIDR', defaultValue: '10.0.2.0/24', description: 'subnet2 cidr')
    string(name: 'InstanceType', defaultValue: 't2.micro', description: 'type of instance')
    string(name: 'AMIID', defaultValue: 'ami-0866a3c8686eaeeba', description: 'amiid')
    string(name: 'VPCName', defaultValue: 'my-vpc', description: 'vpcname')
 }
 stages {
    stage('Checkout Code') {
      steps {
        checkout scmGIT(branches: [[name:'*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'git-credentials', url: 'https://github.com/satwika540/Cloudformation.git']])
     }
    }
    stage('Check app version') {
       steps {
          script {
             def versionsFile = 'deployed_versions.txt'
             if (fileExists(versionsFile)) {
                def versions = readFile(verisonsFile).split('\n')
                if (versions.contains(params.AppVersion)) {
                    error "Application version ${params.AppVersion} already exists! Failing Build."
                } else {
                    echo "Application ${params.AppVersion} is new. Proceeding ..."
                }
             } else {
                 echo "No version file found assuing first deployment"
             }
          }
       }
    }
    stage('Validate Cloudformation Template') {
      steps {
         script {
            echo "Validating template"
         }
         cloudformationCreateChangeSet(
             stackName: "MyApp-stack-${params.Environment}-1",
             changeSetName: "changeset-Validation-${params.GitCommitId}",
             pollInterval: 1000,
             pathToTemplate: 'MyAWSStack.yaml',
             awsRegion: "${env.AWS_REGION}",
             parameters: [
                 ["ParameterKey": AppVersion, "ParameterValue": params.AppVersion],
                 ["ParameterKey": GitCommitId, "ParameterValue": params.GitCommitId]
             ],
             credentialsId: "aws-${params.Environment}-credentials",
             validateOnly: true
         )
      }
    }
    stage('Record App Version') {
       steps {
          script {
              def versionsFile = 'deployed_versions.txt'
              writeFile file: versionsFile, text: "${params.AppVersion}\n", append: true
          }
      }
   }
   stage('Trigger Deployment Pipeline') {
       steps {
          build job: 'deploy-pipeline', wait: false, parameters: [
              string(name: 'AppVersion', value: params.AppVersion),
              string(name: 'GitCommitId', value: params.GitCommitId),
              string(name: 'Environment', value: params.ENV)
              string(name: 'VpcCIDR',  value: params.vpcCIDR)
    	      string(name: 'Subnet1CIDR', value: params.Subnet1CIDR)
              string(name: 'Subnet2CIDR', value: params.Subnet2CIDR)
              string(name: 'InstanceType',value: params.InstanceType)
              string(name: 'AMIID', value: params.AMIID)
              string(name: 'VPCName', value: params.VPCName)
         ]
       }
   }
 }
}
