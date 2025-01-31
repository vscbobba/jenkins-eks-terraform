pipeline {
    agent any

    environment {
        TF_VAR_region = 'us-east-1'  // Correct AWS region
        CLUSTER_NAME = 'HR-dev-eksdemo1'  // EKS cluster name
    }

    parameters {
        choice(name: 'ACTION', choices: ['apply', 'destroy'], description: 'Choose the Terraform action to perform')
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout the repository containing your Terraform scripts
                    git url: 'https://github.com/vscbobba/jenkins-eks-terraform', branch: 'master'
                }
            }
        }

        stage('Terraform Init') {
            steps {
                script {
                    // Use AWS credentials for Terraform
                    withCredentials([aws(credentialsId: 'AWS_ID')]) {
                        // Initialize Terraform using the dev-EKS.config file
                        sh 'terraform init -backend-config=dev-EKS.config'
                    }
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                script {
                    // Run terraform plan using the terraform.tfvars file
                    withCredentials([aws(credentialsId: 'AWS_ID')]) {
                        sh 'terraform plan -var-file=terraform.tfvars -out=tfplan'
                    }
                }
            }
        }

        stage('Terraform Action') {
            steps {
                script {
                    withCredentials([aws(credentialsId: 'AWS_ID')]) {
                        // Conditional logic to either apply or destroy
                        if (params.ACTION == 'apply') {
                            // Apply the Terraform plan if the selected action is 'apply'
                            sh 'terraform apply -var-file=terraform.tfvars -auto-approve'
                        } else if (params.ACTION == 'destroy') {
                            // Destroy the infrastructure if the selected action is 'destroy'
                            sh 'terraform destroy -var-file=terraform.tfvars -auto-approve'
                            // Exit the pipeline immediately after destroy
                            currentBuild.result = 'SUCCESS'  // Mark the build as successful
                            echo "Terraform destroy completed. Exiting the pipeline."
                            return  // Exit the pipeline after destroy
                        }
                    }
                }
            }
        }

        stage('Update aws-auth ConfigMap with Separate AWS Keys') {
            when {
                expression {
                    // Only run this stage if the action was not a 'destroy'
                    return params.ACTION != 'destroy'
                }
            }
            steps {
                script {
                    // Use AWS credentials to apply aws-auth-config.yml (for updating the aws-auth ConfigMap)
                    withCredentials([aws(credentialsId: 'AWS_ID')]) {
                        // Ensure kubectl is configured to use the correct EKS cluster
                        sh '''
                        # Get kubeconfig for the EKS cluster using the new set of credentials (AWS_ID)
                        aws eks --region ${TF_VAR_region} update-kubeconfig --name ${CLUSTER_NAME}

                        # Apply the aws-auth.yml to update the aws-auth ConfigMap
                        kubectl apply -f aws-auth-config.yml
                        '''
                    }
                }
            }
        }

        stage('Post-apply Cleanup') {
            when {
                expression {
                    // Only run this stage if the action was not a 'destroy'
                    return params.ACTION != 'destroy'
                }
            }
            steps {
                script {
                    // Clean up Terraform-related files, such as state and modules
                    sh 'rm -rf .terraform'
                }
            }
        }
    }

    post {
        success {
            echo "Terraform action completed successfully!"
        }
        failure {
            echo "Terraform action failed."
        }
    }
}