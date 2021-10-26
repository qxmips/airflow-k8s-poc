# Managing secrets on Kubernetes Research task

## Issue: 
  the customer stores secrets in config files   along with the code in github. 

## Solution:
    1. option. Use bitmani sealed  secrets operator - https://github.com/bitnami-labs/sealed-secrets . With the operator you can store the secrets in github in an encrypted way  which only operator pod can decrypt using  its tls certificate that is created during operator deployment.    Then  we would reconfigure applications to  mount the kubernetes secrets.

    2. option.  Use AWS secrets manager to store the secrets and secrets provider https://docs.aws.amazon.com/secretsmanager/latest/userguide/integrating_csi_driver.html 

    3.  If the customer uses Terrafrom code with kubernetes provider to deploy workloads on kuberneres , there is also an option to template the config files with terraform and provide secret values  as sensitive variables thought tfvars, or fetch them from a vault or aws secrets manager.
  