# DevSecOps

## Best Practice #1: Build Your Own Key Vault Securely  
### FileName: Own-Key-Vault.md
Goal #1: Securely pass private key to GO Build Pipeline
Goal #2: Securely integrate Jenkins pipeline, docker build 

### Problem to resolve: 
Our project has multiple modules built by GO, in order to build them by Jenkins pipeline, we need to pass the value of Private key to Dockerfile as Build ARG to access private repositories, which is insecure.

### Solution:

#1: Build a tmp key vault container can be accessed from inside the Jenkins server, which allows dockerfile to download the key file during build and kill this tmp container right after build. 

#2: Always use multi-stage build, and access this tmp key server only at the first stage, then copy the build to next stage and remove this first stage container.

## Best Practice #2: Use AWS S3 as Key Vault

Refer to below post for details. Pay attention to below points to further make sure it's really secure:

* Apply S3 policy for that bucket to restrict access via specified IP and User or Role only

* Grant role to that EC2 (say Jenkins)

https://medium.com/@nd1292/aws-configure-for-docker-containers-with-or-without-jenkins-b4c1443a15c9
