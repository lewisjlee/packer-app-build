pipeline {
    agent any
    
    environment {
        AWS_REGION = 'ap-northeast-2'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build AMI and Generate amivar.tf') {
            steps {
                script {
                    sh '''
                        # Packer 플러그인 설치
                        packer init packer-demo.json
                        
                        # AMI 빌드 실행
                        echo "Building AMI with Packer..."
                        ARTIFACT=$(packer build -machine-readable packer-demo.json | awk -F, '$0 ~/artifact,0,id/ {print $6}')
                        echo "Packer output: $ARTIFACT"
                        
                        # AMI ID 추출
                        AMI_ID=$(echo $ARTIFACT | cut -d ':' -f2)
                        echo "AMI ID: ${AMI_ID}"
                        
                        # amivar.tf 생성 및 S3 업로드
                        echo "writing amivar.tf and uploading it to s3"
                        echo 'variable "APP_INSTANCE_AMI" { default = "'${AMI_ID}'" }' > amivar.tf
                        S3_BUCKET=$(aws s3 ls --region $AWS_REGION | grep terraform-state | tail -n1 | cut -d ' ' -f3)
                        aws s3 cp amivar.tf s3://${S3_BUCKET}/amivar.tf --region $AWS_REGION
                        
                        echo "Generated amivar.tf with AMI ID: ${AMI_ID}"
                        echo "Uploaded to S3 bucket: ${S3_BUCKET}"
                    '''
                }
            }
        }
    }
    
    post {
        success {
            echo 'AMI 빌드 및 amivar.tf 생성이 성공적으로 완료되었습니다!'
        }
        failure {
            echo 'AMI 빌드 또는 amivar.tf 생성 중 오류가 발생했습니다.'
        }
    }
} 