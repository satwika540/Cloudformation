pipeline {
    agent any 
    environment {
      COMMIT_AUTHOR = ''
      COMMIT_ID = ''
      COMMIT_TAG = ''
      TARGET_ENV = ''
    }
    parameters {
        string(name: 'VpcCIDR', defaultValue: '10.0.0.0/16', description: 'CIDR OF VPC')
        string(name: 'Subnet1CIDR', defaultValue: '10.0.1.0/24', description: 'CIDR Block for subnet1')
        string(name: 'Subnet2CIDR', defaultValue: '10.0.2.0/24', description: 'CIDR Block for subnet2')
        string(name: 'InstanceType', defaultValue: 't2.micro', description: 'Instance type of EC2')
        string(name: 'AMIId', defaultValue: 'ami-0866a3c8686eaeeba', description: 'Ami required to launch the instance')
    }
       
    stages {
       stage ('Determine Environment') {
          steps {
              script {
                  def branch = env.BRANCH_NAME
                  if (branch == 'dev') {
                      env.TARGET_ENV = 'dev'
                  } else if (branch == 'tst') {
                      env.TARGET_ENV = 'tst'
                  } else if (branch == 'prod') {
                      env.TARGET_ENV = prod
                  } else {
                      error("Unknown branch name: ${branch}")
                  }
              }
          }
       }        
       stage ('Get Commit Info') {
          steps {
              script {
                  env.COMMIT_AUTHOR = sh(script: 'git log -1 --pretty=format:"%an"', returnStdout: true).trim()
                  env.COMMIT_ID = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
              }
          }
       }
       stage ('Validate Cloudformation Template') {
          steps {
              script {
                  echo "Validating cloudformation template for ${env.TARGET_ENV} environment..."
                  withAWS(credentials: "aws-dev-credentials") {
                      cfnValidate(
                          file: 'MyAWSStack.yaml'
                      )
                  }
              }
          }
       }
       stage('Post-Validation') {
           steps {
               script {
                  echo "Validation is successful for commit ${env.COMMIT_ID} by ${env.COMMIT_AUTHOR} on branch ${env.TARGET_ENV}."
               }
           }
       }
       stage ('Trigger Deploy Pipeline') {
           steps {
               script {
                   echo "Triggerring DeployPipeline for environment: ${env.TARGET_ENV}"
                   build job: 'deploy-pipeline', parameters: [
                       string(name: 'VpcCIDR', value: params.VpcCIDR),
                       string(name: 'Subnet1CIDR', value: params.Subnet1CIDR),
                       string(name: 'Subnet2CIDR', value: params.Subnet2CIDR),
                       string(name: 'InstanceType', value: params.InstanceType),
                       string(name: 'TARGET_ENV', value: env.TARGET_ENV),
                       string(name: 'COMMIT_AUTHOR', value: env.COMMIT_AUTHOR),
                       string(name: 'COMMIT_ID', value: env.COMMIT_ID),
                       string(name: 'AMIId', Value: params.AMIId)
                   ]
               }
           }
       }
   }
}
