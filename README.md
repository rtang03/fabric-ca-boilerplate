https://hyperledger-fabric.readthedocs.io/en/release-1.4/enable_tls.html

### start tls-ca at :7052
```shell script
docker-compose up ca-tls
```

### tls-ca cert  
- Create directory `tls-ca/crypto`. `tls-ca` is the tls-ca client.
- Copy the trusted root certificate for the TLS CA from `tls/ca/crypto/ca-cert.pem` 
to `tls-ca/crypto/tls-ca-cert.pem`. 

### enroll the TLS CA admin and then register identities  
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/tls-ca/crypto/tls-ca-cert.pem  && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/tls-ca/admin  && \
fabric-ca-client enroll -d -u https://tls-ca-admin:tls-ca-adminpw@0.0.0.0:7052 && \
fabric-ca-client register -d --id.name peer1-org1 --id.secret peer1PW --id.type peer -u https://0.0.0.0:7052 && \
fabric-ca-client register -d --id.name peer2-org1 --id.secret peer2PW --id.type peer -u https://0.0.0.0:7052 && \
fabric-ca-client register -d --id.name peer1-org2 --id.secret peer1PW --id.type peer -u https://0.0.0.0:7052 && \
fabric-ca-client register -d --id.name peer2-org2 --id.secret peer2PW --id.type peer -u https://0.0.0.0:7052 && \
fabric-ca-client register -d --id.name orderer1-org0 --id.secret ordererPW --id.type orderer -u https://0.0.0.0:7052
```
Add rca-org0 service
Restart docker-compose :7053
Open a new terminal

### Setup Orderer Org CA  
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/ca/admin && \
fabric-ca-client enroll -d -u https://rca-org0-admin:rca-org0-adminpw@0.0.0.0:7053 && \
fabric-ca-client register -d --id.name orderer1-org0 --id.secret ordererpw --id.type orderer -u https://0.0.0.0:7053 && \
fabric-ca-client register -d --id.name admin-org0 --id.secret org0adminpw --id.type admin --id.attrs "hf.Registrar.Roles=client,hf.Registrar.Attributes=*,hf.Revoker=true,hf.GenCRL=true,admin=true:ecert,abac.init=true:ecert" -u https://0.0.0.0:7053
```

Add rca-org1 service
Restart docker-compose :7054
Open a new terminal

### Setup Org1’s CA  
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/ca/admin && \
fabric-ca-client enroll -d -u https://rca-org1-admin:rca-org1-adminpw@0.0.0.0:7054 && \
fabric-ca-client register -d --id.name peer1-org1 --id.secret peer1PW --id.type peer -u https://0.0.0.0:7054 && \
fabric-ca-client register -d --id.name peer2-org1 --id.secret peer2PW --id.type peer -u https://0.0.0.0:7054 && \
fabric-ca-client register -d --id.name admin-org1 --id.secret org1AdminPW --id.type user -u https://0.0.0.0:7054 && \
fabric-ca-client register -d --id.name user-org1 --id.secret org1UserPW --id.type user -u https://0.0.0.0:7054
```

Add rca-org2 service
Restart docker-compose :7055
Open a new terminal

### Setup Org1’s CA  
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org2/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org2/ca/admin && \
fabric-ca-client enroll -d -u https://rca-org2-admin:rca-org2-adminpw@0.0.0.0:7055 && \
fabric-ca-client register -d --id.name peer1-org2 --id.secret peer1PW --id.type peer -u https://0.0.0.0:7055 && \
fabric-ca-client register -d --id.name peer2-org2 --id.secret peer2PW --id.type peer -u https://0.0.0.0:7055 && \
fabric-ca-client register -d --id.name admin-org2 --id.secret org2AdminPW --id.type user -u https://0.0.0.0:7055 && \
fabric-ca-client register -d --id.name user-org2 --id.secret org2UserPW --id.type user -u https://0.0.0.0:7055
```

create `org1/peer1/assets/ca`  
copy `org1/ca/crypto/ca-cert.pem` to `org1/peer1/assets/ca/org1-ca-cert.pem`  
open a new terminal

### Enrol peer1-org1
```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/peer1 && \
export FABRIC_CA_CLIENT_MSPDIR=msp
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/peer1/assets/ca/org1-ca-cert.pem && \
fabric-ca-client enroll -d -u https://peer1-org1:peer1PW@0.0.0.0:7054
```

create `org1/peer1/assets/tls-ca`  
copy `tls/ca/crypto/ca-cert.pem` to `org1/peer1/assets/tls-ca/tls-ca-cert.pem` 
open a new terminal

### Enrol tls-ca at peer1-org1
```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/peer1 && \
export FABRIC_CA_CLIENT_MSPDIR=tls-msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/peer1/assets/tls-ca/tls-ca-cert.pem  && \
fabric-ca-client enroll -d -u https://peer1-org1:peer1PW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts peer1-org1
```

rename key in `org1/peer1/tls-msp/keystore` to `key.pem`
create `org1/peer2/assets/ca`  
copy `org1/ca/crypto/ca-cert.pem` to `org1/peer2/assets/ca/org1-ca-cert.pem`  
Open a new terminal

### Enrol peer2-org1
```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca//org1/peer2 && \
export FABRIC_CA_CLIENT_MSPDIR=msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca//org1/peer2/assets/ca/org1-ca-cert.pem && \
fabric-ca-client enroll -d -u https://peer2-org1:peer2PW@0.0.0.0:7054
```

create `org1/peer2/assets/tls-ca`  
copy `tls/ca/crypto/ca-cert.pem` to `org1/peer2/assets/tls-ca/tls-ca-cert.pem` 
open a new terminal

### Enrol tls-ca at peer2-org1
```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca//org1/peer2 && \
export FABRIC_CA_CLIENT_MSPDIR=tls-msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca//org1/peer2/assets/tls-ca/tls-ca-cert.pem && \
fabric-ca-client enroll -d -u https://peer2-org1:peer2PW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts peer2-org1
```

rename key in `org1/peer1/tls-msp/keystore` to `key.pem`

### Enrol org1 Admin
```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/admin && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/peer1/assets/ca/org1-ca-cert.pem && \
export FABRIC_CA_CLIENT_MSPDIR=msp && \
fabric-ca-client enroll -d -u https://admin-org1:org1AdminPW@0.0.0.0:7054
```

### Create admincerts
```shell script
mkdir /Users/tangross/dev/2019/fabric-ca/org1/peer1/msp/admincerts
cp /Users/tangross/dev/2019/fabric-ca/org1/admin/msp/signcerts/cert.pem /Users/tangross/dev/2019/fabric-ca/org1/peer1/msp/admincerts/org1-admin-cert.pem
```

copy admincerts to peer2 as well

### Launch org1  
Add peer1-org1 and peer2-org1 services
Restart docker-compose: 7051 and 8051

create `org2/peer1/assets/ca`  
copy `org2/ca/crypto/ca-cert.pem` to `org2/peer1/assets/ca/org2-ca-cert.pem` 
open a new terminal

### Enroll peer1-org2
```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org2/peer1 && \
export FABRIC_CA_CLIENT_MSPDIR=msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org2/peer1/assets/ca/org2-ca-cert.pem && \
fabric-ca-client enroll -d -u https://peer1-org2:peer1PW@0.0.0.0:7055
```

create `org2/peer1/assets/tls-ca`  
copy `tls/ca/crypto/ca-cert.pem` to `org2/peer1/assets/tls-ca/tls-ca-cert.pem` 
open a new terminal

### Enroll peer1-org2 tls-ca
```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org2/peer1 && \
export FABRIC_CA_CLIENT_MSPDIR=tls-msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org2/peer1/assets/tls-ca/tls-ca-cert.pem && \
fabric-ca-client enroll -d -u https://peer1-org2:peer1PW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts peer1-org2
```
rename key in `org2/peer1/tls-msp/keystore` to `key.pem`
create `org2/peer2/assets/ca`  
copy `org2/ca/crypto/ca-cert.pem` to `org2/peer2/assets/ca/org2-ca-cert.pem` 
Open a new terminal  

### Enroll peer2-org2
```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org2/peer2 && \
export FABRIC_CA_CLIENT_MSPDIR=msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org2/peer2/assets/ca/org2-ca-cert.pem && \
fabric-ca-client enroll -d -u https://peer2-org2:peer2PW@0.0.0.0:7055
```

create `org2/peer2/assets/tls-ca`  
copy `tls/ca/crypto/ca-cert.pem` to `org2/peer2/assets/tls-ca/tls-ca-cert.pem` 
open a new terminal

### Enroll peer2-org2 tls-ca
```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org2/peer2 && \
export FABRIC_CA_CLIENT_MSPDIR=tls-msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org2/peer2/assets/tls-ca/tls-ca-cert.pem && \
fabric-ca-client enroll -d -u https://peer2-org2:peer2PW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts peer2-org2
```

rename key in `org2/peer1/tls-msp/keystore` to `key.pem`

### Enroll org2 admin
```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org2/admin && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org2/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_MSPDIR=msp && \
fabric-ca-client enroll -d -u https://admin-org2:org2AdminPW@0.0.0.0:7055
```

### Create org2 admincerts
```shell script
mkdir /Users/tangross/dev/2019/fabric-ca/org2/peer1/msp/admincerts
cp /Users/tangross/dev/2019/fabric-ca/org2/admin/msp/signcerts/cert.pem /Users/tangross/dev/2019/fabric-ca/org2/peer1/msp/admincerts/org2-admin-cert.pem
```

create org2 admincert for peer2
create `org0/orderer/assets/ca`  
copy `org0/ca/crypto/ca-cert.pem` to `org0/orderer/assets/ca/org0-ca-cert.pem` 

### Enrol orderer  
```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/orderer && \
export FABRIC_CA_CLIENT_MSPDIR=msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/orderer/assets/ca/org0-ca-cert.pem && \
fabric-ca-client enroll -d -u https://orderer1-org0:ordererpw@0.0.0.0:7053
```

create `org0/orderer/assets/tls-ca`  
copy `tls/ca/crypto/ca-cert.pem` to `org2/peer2/assets/tls-ca/tls-ca-cert.pem` 

### Enrol orderer tls-ca

```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/orderer && \
export FABRIC_CA_CLIENT_MSPDIR=tls-msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/orderer/assets/tls-ca/tls-ca-cert.pem && \
fabric-ca-client enroll -d -u https://orderer1-org0:ordererPW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts orderer1-org0
```

### Create orderer admincerts

```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/admin && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_MSPDIR=msp && \
fabric-ca-client enroll -d -u https://admin-org0:org0adminpw@0.0.0.0:7053
```

```shell script
mkdir /Users/tangross/dev/2019/fabric-ca/org0/orderer/msp/admincerts
cp /Users/tangross/dev/2019/fabric-ca/org0/admin/msp/signcerts/cert.pem /Users/tangross/dev/2019/fabric-ca/org0/orderer/msp/admincerts/orderer-admin-cert.pem
```

create configtx.yaml

create `org0/msp` and its content 
create `org1/msp` and its content
create `org2/msp` and its content

### Genesis Block
```shell script
cd /Users//tangross/dev/2019/fabric-ca/org0
configtxgen -profile OrgsOrdererGenesis -channelID mychannel -outputBlock /Users/tangross/dev/2019/fabric-ca/org0/orderer/genesis.block
configtxgen -profile OrgsChannel -outputCreateChannelTx /Users/tangross/dev/2019/fabric-ca/org0/orderer/channel.tx -channelID mychannel
```

create org1 and org2 CLIs

```shell script
docker exec -it cli-org1 bash
```

```shell script
export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin/msp && \
peer channel create -c mychannel -f /tmp/hyperledger/org1/peer1/assets/channel.tx -o orderer1-org0:7050 \
--outputBlock /tmp/hyperledger/org1/peer1/assets/mychannel.block --tls \
--cafile /tmp/hyperledger/org0/msp/cacerts/org0-ca-cert.pem
```

```shell script
configtxgen -inspectBlock genesis.block
```

