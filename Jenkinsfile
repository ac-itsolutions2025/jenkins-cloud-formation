pipeline {
  agent any

  environment {
    STACK_NAME     = 'acit-custom-vpc'
    TEMPLATE_FILE  = 'custom-vpc.yaml'
    REGION         = 'us-east-2'

    // CIDR parameters (adjust as needed)
    VPC_CIDR           = '10.70.0.0/16'
    PUBLIC1_CIDR       = '10.70.1.0/24'
    PUBLIC2_CIDR       = '10.70.2.0/24'
    PRIVATE1_CIDR      = '10.70.101.0/24'
    PRIVATE2_CIDR      = '10.70.102.0/24'
  }

  options {
    timestamps()
    disableConcurrentBuilds()
  }

  stages {
    stage('Checkout Source') {
      steps {
        echo 'Checking out repo'
        checkout scm
      }
    }

    stage('Install cfn-lint (optional)') {
      steps {
        echo 'Installing linter'
        sh '''
          python3 -m venv .venv
          . .venv/bin/activate
          .venv/bin/pip install --upgrade pip cfn-lint
        '''
      }
    }

    stage('Lint Template') {
      steps {
        echo 'Validating CloudFormation template'
        sh '''
          . .venv/bin/activate
          .venv/bin/cfn-lint --template "$TEMPLATE_FILE"
        '''
      }
    }

    stage('Deploy VPC Stack') {
      steps {
        echo 'Deploying VPC with CloudFormation'
        sh '''
          aws cloudformation deploy \
            --stack-name "$STACK_NAME" \
            --template-file "$TEMPLATE_FILE" \
            --region "$REGION" \
            --capabilities CAPABILITY_NAMED_IAM \
            --parameter-overrides \
              VpcCIDR="$VPC_CIDR" \
              PublicSubnet1CIDR="$PUBLIC1_CIDR" \
              PublicSubnet2CIDR="$PUBLIC2_CIDR" \
              PrivateSubnet1CIDR="$PRIVATE1_CIDR" \
              PrivateSubnet2CIDR="$PRIVATE2_CIDR"
        '''
      }
    }

    stage('Describe Stack Outputs') {
      steps {
        echo '📡 Fetching VPC outputs'
        sh '''
          aws cloudformation describe-stacks \
            --region "$REGION" \
            --stack-name "$STACK_NAME" \
            --query "Stacks[0].Outputs" \
            --output table
        '''
      }
    }
  }

  post {
    always {
      echo '🧹 Cleaning up virtual environment...'
      sh 'rm -rf .venv || true'
    }
    success {
      echo 'VPC stack deployed successfully!'
    }
    failure {
      echo 'Deployment failed. See logs above.'
    }
  }
}
