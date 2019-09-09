uncomment ca-tls, rca-org0, rca-org1, rca-org2

### start cas
```shell script
docker-compose up
```

## SETUP CA
### tls-ca cert  
- Create directory `tls-ca/crypto`. 
- Copy the trusted root certificate for the TLS CA from `tls/ca/crypto/ca-cert.pem` 
to `tls-ca/crypto/tls-ca-cert.pem`. 

### enroll the TLS CA admin => create tls-ca/admin
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/tls-ca/crypto/tls-ca-cert.pem  && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/tls-ca/admin  && \
fabric-ca-client enroll -d -u https://tls-ca-admin:tls-ca-adminpw@0.0.0.0:5052
```

### list affilliation
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/tls-ca/crypto/tls-ca-cert.pem  && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/tls-ca/admin  && \
fabric-ca-client affiliation list
```

### remove default affilliation
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/tls-ca/crypto/tls-ca-cert.pem  && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/tls-ca/admin  && \
fabric-ca-client affiliation remove --force org1 && \
fabric-ca-client affiliation remove --force org2
```

### create affilliation
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/tls-ca/crypto/tls-ca-cert.pem  && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/tls-ca/admin  && \
fabric-ca-client affiliation add com && \
fabric-ca-client affiliation add com.example && \
fabric-ca-client affiliation add com.example.org1 && \
fabric-ca-client affiliation add com.example.org2
```

## SETUP ORDERER
### register tls for orderer
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/tls-ca/crypto/tls-ca-cert.pem  && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/tls-ca/admin  && \
fabric-ca-client register -d --id.name orderer.example.com --id.secret ordererPW --id.affiliation "com.example" --id.type orderer -u https://0.0.0.0:5052 &&
fabric-ca-client register -d --id.name peer0.org1.example.com --id.secret peer1pw --id.type peer --id.affiliation "com.example.org1" -u https://0.0.0.0:5052 &&
fabric-ca-client register -d --id.name peer1.org1.example.com --id.secret peer2pw --id.type peer --id.affiliation "com.example.org1" -u https://0.0.0.0:5052 &&
fabric-ca-client register -d --id.name peer0.org2.example.com --id.secret peer1pw --id.type peer --id.affiliation "com.example.org2" -u https://0.0.0.0:5052 &&
fabric-ca-client register -d --id.name peer1.org2.example.com --id.secret peer2pw --id.type peer --id.affiliation "com.example.org2" -u https://0.0.0.0:5052
```

### enrol orderer example.com 
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/ca/admin && \
fabric-ca-client enroll -d -u https://rca-org0-admin:rca-org0-adminpw@0.0.0.0:5053 
```

### list affilliation for orderer
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/ca/admin && \
fabric-ca-client affiliation list
```

### create affilliation for orderer
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/ca/admin && \
fabric-ca-client affiliation remove --force org1 && \
fabric-ca-client affiliation remove --force org2 && \
fabric-ca-client affiliation add com && \
fabric-ca-client affiliation add com.example && \
fabric-ca-client affiliation add com.example.org1 && \
fabric-ca-client affiliation add com.example.org2
```

### register orderer
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/ca/admin && \
fabric-ca-client register -d --id.name orderer.example.com --id.secret ordererpw --id.affiliation "com.example" --id.type orderer -u https://0.0.0.0:5053 
```

### register orderer admin
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/ca/admin && \
fabric-ca-client register -d --id.name Admin@example.com --id.secret ordererpw --id.type client --id.affiliation "com.example" \
--id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' \
-u https://0.0.0.0:5053
```

// temp
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/ca/admin && \
fabric-ca-client register -d --id.name Admin2@example.com --id.secret ordererpw --id.type admin --id.affiliation "com.example" \
--id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,admin=true:ecert,role=admin:ecert' \
-u https://0.0.0.0:5053
```

create `org0/orderer/assets/ca`  
copy `org0/ca/crypto/ca-cert.pem` to `org0/orderer/assets/ca/org0-ca-cert.pem` 

```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/orderer && \
export FABRIC_CA_CLIENT_MSPDIR=msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/orderer/assets/ca/org0-ca-cert.pem && \
fabric-ca-client enroll -d -u https://orderer.example.com:ordererpw@0.0.0.0:5053
```

create `org0/orderer/assets/tls-ca`  
copy `tls/ca/crypto/tls-ca-cert.pem` to `org2/peer2/assets/tls-ca/tls-ca-cert.pem` 

```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/orderer && \
export FABRIC_CA_CLIENT_MSPDIR=tls-msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/orderer/assets/tls-ca/tls-ca-cert.pem && \
fabric-ca-client enroll -d -u https://orderer.example.com:ordererPW@0.0.0.0:5052 --enrollment.profile tls --csr.hosts orderer.example.com
```

rename key in `org0/orderer/tls-msp/keystore` to `key.pem`

### enrol orderer admin
```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/admin && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_MSPDIR=msp && \
fabric-ca-client enroll -d -u https://Admin@example.com:ordererpw@0.0.0.0:5053
```

// temp
```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/admin2 && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_MSPDIR=msp && \
fabric-ca-client enroll -d -u https://Admin2@example.com:ordererpw@0.0.0.0:5053
```

### create admincerts
```shell script
mkdir /Users/tangross/dev/2019/fabric-ca/org0/orderer/msp/admincerts && \
cp /Users/tangross/dev/2019/fabric-ca/org0/admin/msp/signcerts/cert.pem /Users/tangross/dev/2019/fabric-ca/org0/orderer/msp/admincerts/org0-admin-cert.pem && \
mkdir /Users/tangross/dev/2019/fabric-ca/org0/admin/msp/admincerts && \
cp /Users/tangross/dev/2019/fabric-ca/org0/admin/msp/signcerts/cert.pem /Users/tangross/dev/2019/fabric-ca/org0/admin/msp/admincerts/org0-admin-cert.pem && \
mkdir -p /Users/tangross/dev/2019/fabric-ca/org0/msp/admincerts && \
cp /Users/tangross/dev/2019/fabric-ca/org0/admin/msp/signcerts/cert.pem /Users/tangross/dev/2019/fabric-ca/org0/msp/admincerts/org0-admin-cert.pem
```

## SETUP ORG1
### enrol org1 peers
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/ca/admin && \
fabric-ca-client enroll -d -u https://rca-org1-admin:rca-org1-adminpw@0.0.0.0:5054 
```

### list affilliation for org1
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/ca/admin && \
fabric-ca-client affiliation list
```

### create affilliation for org1
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/ca/admin && \
fabric-ca-client affiliation remove --force org1 && \
fabric-ca-client affiliation remove --force org2 && \
fabric-ca-client affiliation add com && \
fabric-ca-client affiliation add com.example && \
fabric-ca-client affiliation add com.example.org1 && \
fabric-ca-client affiliation add com.example.org2
```

### register org1 peer 0
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/ca/admin && \
fabric-ca-client register -d --id.name peer0.org1.example.com --id.secret peer1pw --id.type peer --id.affiliation "com.example.org1" -u https://0.0.0.0:5054
```

### register org1 peer 1
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/ca/admin && \
fabric-ca-client register -d --id.name peer1.org1.example.com --id.secret peer2pw --id.type peer --id.affiliation "com.example.org1" -u https://0.0.0.0:5054
```

### register org1 admin
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/ca/admin && \
fabric-ca-client register -d --id.name Admin@org1.example.com --id.secret peer1pw --id.type client --id.affiliation "com.example.org1" \
--id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' \
-u https://0.0.0.0:5054
```

//temp
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/ca/admin && \
fabric-ca-client register -d --id.name Admin1@org1.example.com --id.secret peer1pw --id.type admin --id.affiliation "com.example.org1" \
--id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' \
-u https://0.0.0.0:5054
```

//temp
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/ca/admin && \
fabric-ca-client register -d --id.name Admin2@org1.example.com --id.secret peer1pw --id.type admin --id.affiliation "com.example.org1" \
--id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,admin=true:ecert,role=admin:ecert' \
-u https://0.0.0.0:5054
```

### enrol peer0.org1
create `org1/peer0/assets/ca`  
copy `org1/ca/crypto/ca-cert.pem` to `org1/peer1/assets/ca/org1-ca-cert.pem`

```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/peer0 && \
export FABRIC_CA_CLIENT_MSPDIR=msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/peer0/assets/ca/org1-ca-cert.pem && \
fabric-ca-client enroll -d -u https://peer0.org1.example.com:peer1pw@0.0.0.0:5054
```

create `org1/peer0/assets/tls-ca`  
copy `tls/ca/crypto/ca-cert.pem` to `org1/peer0/assets/tls-ca/tls-ca-cert.pem` 

### Enrol tls-ca for peer0.org1
```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/peer0 && \
export FABRIC_CA_CLIENT_MSPDIR=tls-msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/peer0/assets/tls-ca/tls-ca-cert.pem  && \
fabric-ca-client enroll -d -u https://peer0.org1.example.com:peer1pw@0.0.0.0:5052 --enrollment.profile tls --csr.hosts peer0.org1.example.com
```

rename key in `org1/peer0/tls-msp/keystore` to `key.pem`
create `org1/peer1/assets/ca`  
copy `org1/ca/crypto/ca-cert.pem` to `org1/peer1/assets/ca/org1-ca-cert.pem`

```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/peer1 && \
export FABRIC_CA_CLIENT_MSPDIR=msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/peer1/assets/ca/org1-ca-cert.pem && \
fabric-ca-client enroll -d -u https://peer1.org1.example.com:peer2pw@0.0.0.0:5054
```

### Enrol tls-ca for peer0.org1
create `org1/peer1/assets/tls-ca`  
copy `tls/ca/crypto/ca-cert.pem` to `org1/peer2/assets/tls-ca/tls-ca-cert.pem` 

```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/peer1 && \
export FABRIC_CA_CLIENT_MSPDIR=tls-msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/peer1/assets/tls-ca/tls-ca-cert.pem  && \
fabric-ca-client enroll -d -u https://peer1.org1.example.com:peer2pw@0.0.0.0:5052 --enrollment.profile tls --csr.hosts peer1.org1.example.com
```

rename key in `org1/peer2/tls-msp/keystore` to `key.pem`

### enrol org1 admin
```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/admin && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_MSPDIR=msp && \
fabric-ca-client enroll -d -u https://Admin@org1.example.com:peer1pw@0.0.0.0:5054
```

//temp
```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/admin1 && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_MSPDIR=msp && \
fabric-ca-client enroll -d -u https://Admin1@org1.example.com:peer1pw@0.0.0.0:5054
```

//temp
```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/admin2 && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_MSPDIR=msp && \
fabric-ca-client enroll -d -u https://Admin2@org1.example.com:peer1pw@0.0.0.0:5054
```

### create admincerts
```shell script
mkdir -p /Users/tangross/dev/2019/fabric-ca/org1/peer0/msp/admincerts && \
cp /Users/tangross/dev/2019/fabric-ca/org1/admin/msp/signcerts/cert.pem /Users/tangross/dev/2019/fabric-ca/org1/peer0/msp/admincerts/org1-admin-cert.pem && \
mkdir -p /Users/tangross/dev/2019/fabric-ca/org1/peer1/msp/admincerts && \
cp /Users/tangross/dev/2019/fabric-ca/org1/admin/msp/signcerts/cert.pem /Users/tangross/dev/2019/fabric-ca/org1/peer1/msp/admincerts/org1-admin-cert.pem && \
mkdir -p /Users/tangross/dev/2019/fabric-ca/org1/admin/msp/admincerts && \
cp /Users/tangross/dev/2019/fabric-ca/org1/admin/msp/signcerts/cert.pem /Users/tangross/dev/2019/fabric-ca/org1/admin/msp/admincerts/org1-admin-cert.pem && \
mkdir -p /Users/tangross/dev/2019/fabric-ca/org0/msp/admincerts && \
cp /Users/tangross/dev/2019/fabric-ca/org1/admin/msp/signcerts/cert.pem /Users/tangross/dev/2019/fabric-ca/org1/msp/admincerts/org1-admin-cert.pem
```

### prepare msp
```shell script
mkdir -p /Users/tangross/dev/2019/fabric-ca/org0/msp/cacerts && \ 
mkdir -p /Users/tangross/dev/2019/fabric-ca/org1/msp/cacerts && \ 
mkdir -p /Users/tangross/dev/2019/fabric-ca/org0/msp/tlscacerts && \ 
mkdir -p /Users/tangross/dev/2019/fabric-ca/org1/msp/tlscacerts && \ 
mkdir -p /Users/tangross/dev/2019/fabric-ca/org0/msp/admincerts && \ 
mkdir -p /Users/tangross/dev/2019/fabric-ca/org1/msp/admincerts && \ 
cp /Users/tangross/dev/2019/fabric-ca/org0/ca/crypto/ca-cert.pem /Users/tangross/dev/2019/fabric-ca/org0/msp/cacerts/org0-ca-cert.pem && \
cp /Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem /Users/tangross/dev/2019/fabric-ca/org1/msp/cacerts/org1-ca-cert.pem && \
cp /Users/tangross/dev/2019/fabric-ca/tls-ca/crypto/tls-ca-cert.pem /Users/tangross/dev/2019/fabric-ca/org0/msp/tlscacerts/tls-ca-cert.pem && \
cp /Users/tangross/dev/2019/fabric-ca/tls-ca/crypto/tls-ca-cert.pem /Users/tangross/dev/2019/fabric-ca/org1/msp/tlscacerts/tls-ca-cert.pem && \
cp /Users/tangross/dev/2019/fabric-ca/org1/admin/msp/signcerts/cert.pem /Users/tangross/dev/2019/fabric-ca/org1/msp/admincerts/org1-admin-cert.pem
```


copy configtx.yaml to org0

### 
```shell script
configtxgen -profile TwoOrgsOrdererGenesis -channelID syschannel -outputBlock /Users/tangross/dev/2019/fabric-ca/org0/orderer/genesis.block && \
configtxgen -profile TwoOrgsChannel -channelID mychannel -outputCreateChannelTx /Users/tangross/dev/2019/fabric-ca/org0/orderer/channel.tx && \
configtxgen -profile TwoOrgsChannel -channelID mychannel -outputAnchorPeersUpdate /Users/tangross/dev/2019/fabric-ca/org0/orderer/Org1MSPanchors.tx -channelID mychannel -asOrg Org1MSP
#configtxgen -profile TwoOrgsChannel -channelID mychannel -outputAnchorPeersUpdate Org2MSPanchors.tx -channelID mychannel -asOrg Org2MSP

configtxgen -inspectBlock genesis.block
```

uncomment cli and peers
restart docker compose

```shell script
docker exec -it cli-org1 bash
```

### create channel
```shell script
export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin2/msp && \
peer channel create -c mychannel -f /tmp/hyperledger/org1/peer0/assets/channel.tx -o orderer.example.com:7050 \
--outputBlock /tmp/hyperledger/org1/peer0/assets/mychannel.block --tls \
--cafile /tmp/hyperledger/org1/peer0/tls-msp/tlscacerts/tls-0-0-0-0-5052.pem
```

### join channel
```shell script
export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin1/msp && \
peer channel join -b /tmp/hyperledger/org1/peer0/assets/mychannel.block -o orderer.example.com:7050 \
--cafile /tmp/hyperledger/org1/peer0/tls-msp/tlscacerts/tls-0-0-0-0-5052.pem
```

docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin/msp" peer0.org1.example.com peer channel join -b /tmp/hyperledger/org1/peer0/assets/mychannel.block
