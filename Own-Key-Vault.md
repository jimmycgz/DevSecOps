## Best Practice #1: Build Your Own Key Vault Securely  
### FileName: Own-Key-Vault.md
Goal #1: Securely pass private key to GO Build Pipeline
Goal #2: Securely integrate Jenkins pipeline, docker build 

### Problem to resolve: 
Our project has multiple modules built by GO, in order to build them by Jenkins pipeline, we need to pass the value of Private key to Dockerfile as Build ARG to access private repositories, which is insecure.

### Solution:

#1: Build a tmp key vault container can be accessed from inside the Jenkins server, which allows dockerfile to download the key file during build and kill this tmp container right after build. 

#2: Always use multi-stage build, and access this tmp key server only at the first stage, then copy the build to next stage and remove this first stage container.


### Step 1: Save your private key as local file in EC2
```
mkdir -p $HOME/tmp-key
cd $HOME/tmp-key
nano tmp-private-key
sudo chmod 400 
```
### Step 2: Spin up a tmp web service as key vault
```
docker run --name=tmp-key-vault --rm --volume $HOME/tmp-key-folder:/tmp-key busybox httpd -f -p 9999 -h /tmp-key

#Security check, can't telnet 9999
telnet pub-ip 9999
telnet localhost 9999

# Only below works:
Serv_name="tmp-key-vault"
IPS=`(sudo docker inspect ${Serv_name} | egrep -o '([0-9]+\.){3}[2-9]+')`
IP=${IPS:0:10}
echo $IP
telnet $Serv_name 9999
telnet $IP 9999
```
### Step 3: Modify Dockerfile
Replace below line:
```
&& echo "${SSH_PRIVATE_KEY}" > ~/.ssh/id_rsa \
```
with:
```
&& wget -qO ~/.ssh/id_rsa localhost:9999/tmp-key-file \
```

### Step 4: Try Docker Build with Network Arg
```
git clone build-repo from github
git checkout develop

export tag="ssh-test"
docker build -t build-name:${tag} \
                --build-arg BUILD_HASH="`git describe --match=M..R..tCh --always --abbrev=40 --dirty`" \
                --network=container:tmp-key-vault ./
```

### Step 5:  Update Jenkins pipeline

Spin up the tmp key vault container before docker build

Kill this tmp key server right after docker build
