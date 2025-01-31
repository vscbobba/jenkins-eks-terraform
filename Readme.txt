Start a instance from jenkins-EKS-terraform template.

Apply role Jenkins/Ec2-full from GUi.

        login to cli and run:
        aws eks --region ${TF_VAR_region} update-kubeconfig --name ${CLUSTER_NAME}
        aws eks --region us-east-1 update-kubeconfig --name HR-dev-eksdemo1


        # Now perform kubectl commands (using the Jenkins IAM role)
            kubectl get nodes  # Example command to confirm access
