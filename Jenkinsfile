
def aws_region_var = ''

if(BRANCH_NAME ==~ "dev.*") {
    println("Applying Dev")
    aws_region_var = "us-east-1"
    environment = "dev"
}
else if(BRANCH_NAME ==~ "qa.*") {
    println("Applying QA")
    aws_region_var = "us-east-2"
    environment = "qa"
}
else if(BRANCH_NAME == "main") {
    println("Applying Prod")
    aws_region_var = "us-west-2"
    environment = "prod"
}
else {
    error("Branch name did not match")
}

node {
    stage('Pull Repo') {
        git 'https://github.com/ikambarov/packer.git'
    }

    withCredentials([usernamePassword(credentialsId: 'aws_jenkins_key', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
        withEnv(["AWS_REGION=${aws_region_var}", "PACKER_AMI_NAME=apache-${UUID.randomUUID().toString()}"]) {
            stage('Packer Validate') {
                sh 'packer validate apache.json'
            }

            stage('Packer Build') {
                sh 'packer build apache.json'
                ami_id = sh(script: 'packer build apache.json | grep us-east-1 | awk '{print $2}', returnStdout: true)'
            }
             
            stage('Create Instance') {
                build wait: false, job: 'terraform-ec2', parameters: [booleanParam(name: 'terraform_apply', value: true), booleanParam(name: 'terraform_destroy', value: false), string(name: 'environment', value: "${environment}"), string(name: 'ami_id', value: "${ami_id}")]
            }
        }     
    }
}
