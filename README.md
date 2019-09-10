### tls-ca, and all root CA
Uncomment ca-tls, rca-org0, rca-org1, rca-org2 
```shell script
docker-compose up
```
It should create rca-org0, rca-org1, rca-org2.

## SETUP CA
### Prepare tls-ca client  
```shell script
// copy the trusted root certificate for the TLS CA
mkdir -p tls-ca/crypto
cp ./tls/ca/crypto/ca-cert.pem ./tls-ca/crypto/tls-ca-cert.pem
```

### Enroll the tls-ca admin
Below command will create `./tls-ca/admin` directory, which contains tls-ca admin msp.  
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/tls-ca/crypto/tls-ca-cert.pem  && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/tls-ca/admin  && \
fabric-ca-client enroll -d -u https://tls-ca-admin:tls-ca-adminpw@0.0.0.0:5052
```

### Clean up affilliation
```shell script
// Optional step, to remove pre-existing dummy affilliation
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/tls-ca/crypto/tls-ca-cert.pem  && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/tls-ca/admin  && \
fabric-ca-client affiliation remove --force org1 && \
fabric-ca-client affiliation remove --force org2

// list affiliation
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/tls-ca/crypto/tls-ca-cert.pem  && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/tls-ca/admin  && \
fabric-ca-client affiliation list
```

## SETUP ORDERER
### Register tls for orderer  
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/tls-ca/crypto/tls-ca-cert.pem  && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/tls-ca/admin  && \
fabric-ca-client register -d --id.name orderer.example.com --id.secret ordererPW --id.type orderer -u https://0.0.0.0:5052 &&
fabric-ca-client register -d --id.name peer0.org1.example.com --id.secret peer1pw --id.type peer -u https://0.0.0.0:5052 &&
fabric-ca-client register -d --id.name peer1.org1.example.com --id.secret peer2pw --id.type peer -u https://0.0.0.0:5052 &&
fabric-ca-client register -d --id.name peer0.org2.example.com --id.secret peer1pw --id.type peer -u https://0.0.0.0:5052 &&
fabric-ca-client register -d --id.name peer1.org2.example.com --id.secret peer2pw --id.type peer -u https://0.0.0.0:5052
```

### Enrol orderer example.com   
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/ca/admin && \
fabric-ca-client enroll -d -u https://rca-org0-admin:rca-org0-adminpw@0.0.0.0:5053 
```

### Clean up affilliation for orderer
```shell script
// Optional step, to remove pre-existing dummy affilliation
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/ca/admin && \
fabric-ca-client affiliation remove --force org1 && \
fabric-ca-client affiliation remove --force org2 

// list affiliation
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/ca/admin && \
fabric-ca-client affiliation list
```

### Register orderer
```shell script
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/ca/admin && \
fabric-ca-client register -d --id.name orderer.example.com --id.secret ordererpw --id.type orderer -u https://0.0.0.0:5053 
```

### Register orderer admin
```shell script
// register orderer
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/ca/admin && \
fabric-ca-client register -d --id.name Admin@example.com --id.secret ordererpw --id.type client \
--id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,admin=true:ecert,role=admin:ecert,abac.init=true:ecert' \
-u https://0.0.0.0:5053

// prepare assets
mkdir -p /Users/tangross/dev/2019/fabric-ca/org0/orderer/assets/ca
cp /Users/tangross/dev/2019/fabric-ca/org0/ca/crypto/ca-cert.pem /Users/tangross/dev/2019/fabric-ca/org0/orderer/assets/ca/org0-ca-cert.pem
mkdir -p /Users/tangross/dev/2019/fabric-ca/org0/orderer/assets/tls-ca
cp /Users/tangross/dev/2019/fabric-ca/tls-ca/crypto/tls-ca-cert.pem /Users/tangross/dev/2019/fabric-ca/org0/orderer/assets/tls-ca/tls-ca-cert.pem

// create msp
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/orderer && \
export FABRIC_CA_CLIENT_MSPDIR=msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/orderer/assets/ca/org0-ca-cert.pem && \
fabric-ca-client enroll -d -u https://orderer.example.com:ordererpw@0.0.0.0:5053 

// create tls-msp
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/orderer && \
export FABRIC_CA_CLIENT_MSPDIR=tls-msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/orderer/assets/tls-ca/tls-ca-cert.pem && \
fabric-ca-client enroll -d -u https://orderer.example.com:ordererPW@0.0.0.0:5052 --enrollment.profile tls --csr.hosts orderer.example.com
```

rename key file in `org0/orderer/tls-msp/keystore` to `key.pem`

### enrol orderer admin
```shell script
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org0/admin && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org0/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_MSPDIR=msp && \
fabric-ca-client enroll -d -u https://Admin@example.com:ordererpw@0.0.0.0:5053

// create admincerts
mkdir /Users/tangross/dev/2019/fabric-ca/org0/orderer/msp/admincerts && \
cp /Users/tangross/dev/2019/fabric-ca/org0/admin/msp/signcerts/cert.pem /Users/tangross/dev/2019/fabric-ca/org0/orderer/msp/admincerts/org0-admin-cert.pem && \
mkdir /Users/tangross/dev/2019/fabric-ca/org0/admin/msp/admincerts && \
cp /Users/tangross/dev/2019/fabric-ca/org0/admin/msp/signcerts/cert.pem /Users/tangross/dev/2019/fabric-ca/org0/admin/msp/admincerts/org0-admin-cert.pem && \
mkdir -p /Users/tangross/dev/2019/fabric-ca/org0/msp/admincerts && \
cp /Users/tangross/dev/2019/fabric-ca/org0/admin/msp/signcerts/cert.pem /Users/tangross/dev/2019/fabric-ca/org0/msp/admincerts/org0-admin-cert.pem
```

## SETUP ORG1
### Enrol org1 peers
```shell script
// enrol org1 admin
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/ca/admin && \
fabric-ca-client enroll -d -u https://rca-org1-admin:rca-org1-adminpw@0.0.0.0:5054

// Optional step, to remove pre-existing dummy affilliation
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/ca/admin && \
fabric-ca-client affiliation remove --force org1 && \
fabric-ca-client affiliation remove --force org2 

// Optional step
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/ca/admin && \
fabric-ca-client affiliation list
```

### Register org1 peer 0/1
```shell script
// register org1 peer 0/1
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/ca/admin && \
fabric-ca-client register -d --id.name peer0.org1.example.com --id.secret peer1pw --id.type peer -u https://0.0.0.0:5054 && \
fabric-ca-client register -d --id.name peer1.org1.example.com --id.secret peer2pw --id.type peer -u https://0.0.0.0:5054

// register org1 admin
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/ca/admin && \
fabric-ca-client register -d --id.name Admin@org1.example.com --id.secret peer1pw --id.type client \
--id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,admin=true:ecert,role=admin:ecert,abac.init=true:ecert' \
-u https://0.0.0.0:5054
```

### enrol peer0.org1
```shell script
// preparing assets
mkdir -p /Users/tangross/dev/2019/fabric-ca/org1/peer0/assets/ca
cp /Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem /Users/tangross/dev/2019/fabric-ca/org1/peer0/assets/ca/org1-ca-cert.pem
mkdir -p /Users/tangross/dev/2019/fabric-ca/org1/peer0/assets/tls-ca
cp /Users/tangross/dev/2019/fabric-ca/tls-ca/crypto/tls-ca-cert.pem /Users/tangross/dev/2019/fabric-ca/org1/peer0/assets/tls-ca/tls-ca-cert.pem

// enrol peer0.org1
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/peer0 && \
export FABRIC_CA_CLIENT_MSPDIR=msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/peer0/assets/ca/org1-ca-cert.pem && \
fabric-ca-client enroll -d -u https://peer0.org1.example.com:peer1pw@0.0.0.0:5054

// enrol tls-ca for peer0.org1
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/peer0 && \
export FABRIC_CA_CLIENT_MSPDIR=tls-msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/peer0/assets/tls-ca/tls-ca-cert.pem  && \
fabric-ca-client enroll -d -u https://peer0.org1.example.com:peer1pw@0.0.0.0:5052 --enrollment.profile tls --csr.hosts peer0.org1.example.com
``` 

rename key in `org1/peer0/tls-msp/keystore` to `key.pem`

### enrol peer1.org1
```shell script
// preparing assets
mkdir -p /Users/tangross/dev/2019/fabric-ca/org1/peer1/assets/ca
cp /Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem /Users/tangross/dev/2019/fabric-ca/org1/peer1/assets/ca/org1-ca-cert.pem
mkdir -p /Users/tangross/dev/2019/fabric-ca/org1/peer1/assets/tls-ca
cp /Users/tangross/dev/2019/fabric-ca/tls-ca/crypto/tls-ca-cert.pem /Users/tangross/dev/2019/fabric-ca/org1/peer1/assets/tls-ca/tls-ca-cert.pem

// enrol peer1.org1
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/peer1 && \
export FABRIC_CA_CLIENT_MSPDIR=msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/peer1/assets/ca/org1-ca-cert.pem && \
fabric-ca-client enroll -d -u https://peer1.org1.example.com:peer2pw@0.0.0.0:5054 

// enrol tls-ca for peer1.org1
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/peer1 && \
export FABRIC_CA_CLIENT_MSPDIR=tls-msp && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/peer1/assets/tls-ca/tls-ca-cert.pem  && \
fabric-ca-client enroll -d -u https://peer1.org1.example.com:peer2pw@0.0.0.0:5052 --enrollment.profile tls --csr.hosts peer1.org1.example.com
``` 

rename key in `org1/peer1/tls-msp/keystore` to `key.pem`

### enrol org1 admin
```shell script
// enrol org admin
export FABRIC_CA_CLIENT_HOME=/Users/tangross/dev/2019/fabric-ca/org1/admin && \
export FABRIC_CA_CLIENT_TLS_CERTFILES=/Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem && \
export FABRIC_CA_CLIENT_MSPDIR=msp && \
fabric-ca-client enroll -d -u https://Admin@org1.example.com:peer1pw@0.0.0.0:5054

// create admincerts
mkdir -p /Users/tangross/dev/2019/fabric-ca/org1/peer0/msp/admincerts && \
cp /Users/tangross/dev/2019/fabric-ca/org1/admin/msp/signcerts/cert.pem /Users/tangross/dev/2019/fabric-ca/org1/peer0/msp/admincerts/org1-admin-cert.pem && \
mkdir -p /Users/tangross/dev/2019/fabric-ca/org1/peer1/msp/admincerts && \
cp /Users/tangross/dev/2019/fabric-ca/org1/admin/msp/signcerts/cert.pem /Users/tangross/dev/2019/fabric-ca/org1/peer1/msp/admincerts/org1-admin-cert.pem && \
mkdir -p /Users/tangross/dev/2019/fabric-ca/org1/admin/msp/admincerts && \
cp /Users/tangross/dev/2019/fabric-ca/org1/admin/msp/signcerts/cert.pem /Users/tangross/dev/2019/fabric-ca/org1/admin/msp/admincerts/org1-admin-cert.pem && \
mkdir -p /Users/tangross/dev/2019/fabric-ca/org1/msp/admincerts && \
cp /Users/tangross/dev/2019/fabric-ca/org1/admin/msp/signcerts/cert.pem /Users/tangross/dev/2019/fabric-ca/org1/msp/admincerts/org1-admin-cert.pem
```

### Prepare msp
```shell script
mkdir -p /Users/tangross/dev/2019/fabric-ca/org0/msp/cacerts && \ 
mkdir -p /Users/tangross/dev/2019/fabric-ca/org1/msp/cacerts && \ 
mkdir -p /Users/tangross/dev/2019/fabric-ca/org0/msp/tlscacerts && \ 
mkdir -p /Users/tangross/dev/2019/fabric-ca/org1/msp/tlscacerts && \ 
cp /Users/tangross/dev/2019/fabric-ca/org0/ca/crypto/ca-cert.pem /Users/tangross/dev/2019/fabric-ca/org0/msp/cacerts/org0-ca-cert.pem && \
cp /Users/tangross/dev/2019/fabric-ca/org1/ca/crypto/ca-cert.pem /Users/tangross/dev/2019/fabric-ca/org1/msp/cacerts/org1-ca-cert.pem && \
cp /Users/tangross/dev/2019/fabric-ca/tls-ca/crypto/tls-ca-cert.pem /Users/tangross/dev/2019/fabric-ca/org0/msp/tlscacerts/tls-ca-cert.pem && \
cp /Users/tangross/dev/2019/fabric-ca/tls-ca/crypto/tls-ca-cert.pem /Users/tangross/dev/2019/fabric-ca/org1/msp/tlscacerts/tls-ca-cert.pem
```

## Deploy  
### Prepare configtx
```shell script
cp ./configtx.yaml to ./org0/configtx.yaml

cd org0
export FABRIC_CFG_PATH=$PWD && \
configtxgen -profile TwoOrgsOrdererGenesis -channelID syschannel -outputBlock /Users/tangross/dev/2019/fabric-ca/org0/orderer/genesis.block && \
configtxgen -profile TwoOrgsChannel -channelID mychannel -outputCreateChannelTx /Users/tangross/dev/2019/fabric-ca/org0/orderer/channel.tx && \
configtxgen -profile TwoOrgsChannel -channelID mychannel -outputAnchorPeersUpdate /Users/tangross/dev/2019/fabric-ca/org0/orderer/Org1MSPanchors.tx -channelID mychannel -asOrg Org1MSP && \
cp /Users/tangross/dev/2019/fabric-ca/org0/orderer/channel.tx /Users/tangross/dev/2019/fabric-ca/org1/peer0/assets/channel.tx && \
cp /Users/tangross/dev/2019/fabric-ca/org0/orderer/Org1MSPanchors.tx /Users/tangross/dev/2019/fabric-ca/org1/peer0/assets/Org1MSPanchors.tx
#configtxgen -profile TwoOrgsChannel -channelID mychannel -outputAnchorPeersUpdate Org2MSPanchors.tx -channelID mychannel -asOrg Org2MSP
```

### Create and join channel 
uncomment cli and peers, and restart docker-compose

```shell script
// log on to cli
docker exec -it cli-org1 bash

// create channel and join channel peer0.org1
export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin/msp && \
peer channel create -c mychannel -f /tmp/hyperledger/org1/peer0/assets/channel.tx -o orderer.example.com:7050 \
--outputBlock /tmp/hyperledger/org1/peer0/assets/mychannel.block --tls \
--cafile /tmp/hyperledger/org1/peer0/tls-msp/tlscacerts/tls-0-0-0-0-5052.pem 

// join channel for peer0.org1
export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin/msp && \ 
peer channel join -b /tmp/hyperledger/org1/peer0/assets/mychannel.block

// join channel for peer1.org1
export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin/msp peer && \
export CORE_PEER_ADDRESS=peer1.org1.example.com:8051 && \
peer channel join -b /tmp/hyperledger/org1/peer0/assets/mychannel.block

// anchor peer
export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin/msp && \
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051 && \
peer channel update -o orderer.example.com:7050 -c mychannel -f /tmp/hyperledger/org1/peer0/assets/Org1MSPanchors.tx \
--tls --cafile /tmp/hyperledger/org1/peer0/tls-msp/tlscacerts/tls-0-0-0-0-5052.pem  

// install cc to peer0.org1
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051 && \
export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin/msp && \
peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/abac/go

// install cc to peer0.org1
export CORE_PEER_ADDRESS=peer1.org1.example.com:8051 && \
export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin/msp && \
peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/abac/go

// instantiate
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051 && \
export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin/msp && \
peer chaincode instantiate -C mychannel -n mycc -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -o orderer.example.com:7050 --tls \
--cafile /tmp/hyperledger/org1/peer0/tls-msp/tlscacerts/tls-0-0-0-0-5052.pem  

export CORE_PEER_ADDRESS=peer0.org1.example.com:7051 && \
export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin/msp && \
peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'
// return 100

export CORE_PEER_ADDRESS=peer0.org1.example.com:7051 && \
export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin/msp && \
peer chaincode invoke -C mychannel -n mycc -c '{"Args":["invoke","a","b","10"]}' --tls --cafile /tmp/hyperledger/org1/peer0/tls-msp/tlscacerts/tls-0-0-0-0-5052.pem

peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'
// return 90
```

## Troubleshooting
```shell script
// decode cert
openssl x509 -in key.pem -text -noout

// decode mychannel.block
cd org1/assets
configtxlator proto_decode --type common.Block --input mychannel.block --output mychannel.block.json

// inpsect genesis block
cd org0
export FABRIC_CFG_PATH=$PWD && \
configtxgen -inspectBlock ./orderer/genesis.block

// inspect channel.tx
cd org0
export FABRIC_CFG_PATH=$PWD && \
configtxgen  -inspectChannelCreateTx ./orderer/channel.tx

// inside block, decode the individual cert 
echo "LS0tLS1CRUdJTiBDRVJUSU....tLS0tCg==" |base64 -D >a.cert && \
openssl x509 -in a.cert -text

// list channel
peer channel list 
```

