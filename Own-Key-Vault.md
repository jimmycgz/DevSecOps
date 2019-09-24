# Step 1: Save your private key as local file in EC2
mkdir -p $HOME/tmp-key
cd $HOME/tmp-key
nano tmp-private-key
sudo chmod 400 

# Step 2: Spin up a tmp web service as key vault
docker run --name=tmp-secrets-server --rm --volume $HOME/tmp-key:/tmp-key busybox httpd -f -p 9999 -h /tmp-key

#Security check, can't telnet 9999
telnet 35.182.191.212 9999
telnet localhost 9999

# Only below works:
Serv_name="tmp-secrets-server"
IPS=`(sudo docker inspect ${Serv_name} | egrep -o '([0-9]+\.){3}[2-9]+')`
IP=${IPS:0:10}
echo $IP
telnet $Serv_name 9999
telnet $IP 9999

# Step 3: Modify Dockerfile
Replace below line:
 && echo "${SSH_PRIVATE_KEY}" > ~/.ssh/id_rsa \
with:
 && wget -qO ~/.ssh/id_rsa localhost:9999/tmp-private-key \
 
# Step 4: Try Docker Build with Network Arg

git checkout develop

export tag="ssh-test"
docker build -t docker.api.deepaffex.ai/dfx-api-v3:${tag} \
                --build-arg BUILD_HASH="`git describe --match=NeVeRmAtCh --always --abbrev=40 --dirty`" \
                --network=container:tmp-secrets-server ./
