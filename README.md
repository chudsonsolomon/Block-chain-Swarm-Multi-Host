# Block-chain-Swarm-Multi-Host
Run command on  Host1:
------------------------
docker swarm init
docker network create --attachable --driver overlay my_network

Run command on  Host2:
------------------------
docker swarm join --token SWMTKN-1-0nwxf6rqlx6tjec5emxywl19s0dn7iuwgc1736xhz14jx8mx1f-elsrc31y6nx8uuhi6b3x2nm2j Host1-ip:2377

Host1:
------
# Replace crypto-config.yaml and configtx.yaml file as per your organization Name.
   - ./cryptogen generate --config=crypto-config.yaml
   - ./configtxgen -profile mychannel -outputCreateChannelTx ./composer-channel.tx -channelID mychannel
   - ./configtxgen -profile ComposerOrdererGenesis -outputBlock ./composer-genesis.block

docker run -it -d --network="my_network" -p 27054:7054 -e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=my_network -e FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server -e FABRIC_CA_SERVER_CA_NAME=ca.org1.example.com -v /opt/swarm-test/fabric-scripts/hlfv12/composer/crypto-config/peerOrganizations/org1.example.com/ca/:/etc/hyperledger/fabric-ca-server-config --name=ca.org1.example.swarm.com hyperledger/fabric-ca:1.2.1 sh -c 'fabric-ca-server start --ca.certfile /etc/hyperledger/fabric-ca-server-config/ca.org1.example.com-cert.pem --ca.keyfile /etc/hyperledger/fabric-ca-server-config/*_sk -b admin:adminpw -d'

docker run -it -d --network="my_network" -p 27050:7050 -e ORDERER_GENERAL_LOGLEVEL=debug -e ORDERER_GENERAL_LISTENADDRESS=0.0.0.0 -e ORDERER_GENERAL_GENESISMETHOD=file -e ORDERER_GENERAL_GENESISFILE=/etc/hyperledger/configtx/composer-genesis.block -e ORDERER_GENERAL_LOCALMSPID=OrdererMSP -e ORDERER_GENERAL_LOCALMSPDIR=/etc/hyperledger/msp/orderer/msp -v /opt/swarm-test/fabric-scripts/hlfv12/composer/:/etc/hyperledger/configtx -v /opt/swarm-test/fabric-scripts/hlfv12/composer/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp:/etc/hyperledger/msp/orderer/msp --workdir=/opt/gopath/src/github.com/hyperledger/fabric --name=orderer.example.swarm.com hyperledger/fabric-orderer:1.2.1 sh -c 'orderer'

docker run -it -d --network="my_network" -p 25984:5984 -e 'DB_URL: <http://Host1-IP:25984/member_db'> --name=couchdb-swarm hyperledger/fabric-couchdb:0.4.10

docker run -it -d --network="my_network" -p 27051:7051 -p 27053:7053 -e CORE_LOGGING_LEVEL=debug -e CORE_CHAINCODE_LOGGING_LEVEL=DEBUG -e CORE_CHAINCODE_STARTUPTIMEOUT=900s -e CORE_CHAINCODE_EXECUTETIMEOUT=900s -e CORE_CHAINCODE_DEPLOYTIMEOUT=900s -e CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock -e CORE_PEER_ID=peer0.org1.example-swarm.com -e CORE_PEER_ADDRESS=peer0.org1.example-swarm.com:7051 -e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=my_network -e CORE_PEER_LOCALMSPID=org1MSP -e CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/msp -e CORE_LEDGER_STATE_STATEDATABASE=CouchDB -e CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=Host1-IP:25984 -v /var/run/:/host/var/run/ -v /opt/swarm-test/fabric-scripts/hlfv12/composer:/etc/hyperledger/configtx -v /opt/swarm-test/fabric-scripts/hlfv12/composer/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp:/etc/hyperledger/peer/msp -v /opt/swarm-test/fabric-scripts/hlfv12/composer/crypto-config/peerOrganizations/org1.example.com/users:/etc/hyperledger/msp/users --workdir=/opt/gopath/src/github.com/hyperledger/fabric --name=peer0.org1.example-swarm.com hyperledger/fabric-peer:1.2.1 sh -c 'peer node start'

docker exec peer0.org1.example-swarm.com peer channel create -o Host1-IP:27050 -c mychannel -f /etc/hyperledger/configtx/composer-channel.tx

docker exec -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp" peer0.org1.example-swarm.com peer channel join -b mychannel.block


Host2:
---------
# Replace crypto-config.yaml and configtx.yaml file as per your organization Name.

./cryptogen generate --config=crypto-config.yaml

docker run -it -d --network="my_network" -p 26984:5984 -e 'DB_URL: <http://Host2-IP:26984/member_db'> --name=couchdb-swarm2 hyperledger/fabric-couchdb:0.4.10


docker run -it -d --network="my_network" -p 27054:7054 -e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=my_network -e FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server -e FABRIC_CA_SERVER_CA_NAME=ca.org2.example.com -v /opt/swarm-test/fabric-scripts/hlfv12/composer/crypto-config/peerOrganizations/org2.example.com/ca/:/etc/hyperledger/fabric-ca-server-config --name=ca.org2.example.swarm.com hyperledger/fabric-ca:1.2.1 sh -c 'fabric-ca-server start --ca.certfile /etc/hyperledger/fabric-ca-server-config/ca.org2.example.com-cert.pem --ca.keyfile /etc/hyperledger/fabric-ca-server-config/*_sk -b admin:adminpw -d'

docker run -it -d --network="my_network" -p 27051:7051 -p 27053:7053 -e CORE_LOGGING_LEVEL=debug -e CORE_CHAINCODE_LOGGING_LEVEL=DEBUG -e CORE_CHAINCODE_STARTUPTIMEOUT=900s -e CORE_CHAINCODE_EXECUTETIMEOUT=900s -e CORE_CHAINCODE_DEPLOYTIMEOUT=900s -e CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock -e CORE_PEER_ID=peer0.org2.example-swarm.com -e CORE_PEER_ADDRESS=peer0.org2.example-swarm.com:7051 -e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=my_network -e CORE_PEER_LOCALMSPID=org2MSP -e CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/msp -e CORE_LEDGER_STATE_STATEDATABASE=CouchDB -e CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=Host2-IP:26984 -v /var/run/:/host/var/run/ -v /opt/swarm-test/fabric-scripts/hlfv12/composer:/etc/hyperledger/configtx -v /opt/swarm-test/fabric-scripts/hlfv12/composer/crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp:/etc/hyperledger/peer/msp -v /opt/swarm-test/fabric-scripts/hlfv12/composer/crypto-config/peerOrganizations/org2.example.com/users:/etc/hyperledger/msp/users --workdir=/opt/gopath/src/github.com/hyperledger/fabric --name=peer0.org2.example-swarm.com hyperledger/fabric-peer:1.2.1 sh -c 'peer node start'

Host2:
-------------
./configtxgen -printOrg org2MSP > org2.json
scp org2.json root@<Replace-Host1-IP>:/opt/swarm-test/


Host1:
-------------
   - docker cp configtxlator peer0.org1.example-swarm.com:/opt/gopath/src/github.com/hyperledger/fabric/
   - docker cp  org2.json peer0.org1.example-swarm.com:/opt/gopath/src/github.com/hyperledger/fabric/
   - docker exec -it peer0.org1.example-swarm.com /bin/bash
   - peer channel fetch config config_block.pb -o orderer.example.swarm.com:7050 -c mychannel
   - apt-get update
   - apt-get install jq -y
   - ./configtxlator proto_decode --input config_block.pb --type common.Block | jq .data.data[0].payload.data.config > config.json
   - jq -s '.[0] * {"channel_group":{"groups":{"Application":{"groups": {"org2MSP":.[1]}}}}}' config.json org2.json >                        modified_config.json
   - ./configtxlator proto_encode --input config.json --type common.Config --output config.pb
   - ./configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb
   - ./configtxlator compute_update --channel_id mychannel --original config.pb --updated modified_config.pb --output org2_update.pb
   - ./configtxlator proto_decode --input org2_update.pb --type common.ConfigUpdate | jq . > org2_update.json
   - echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}},"data":{"config_update":'$(cat                        org2_update.json)'}}}' | jq . > org2_update_in_envelope.json
   - ./configtxlator proto_encode --input org2_update_in_envelope.json --type common.Envelope --output org2_update_in_envelope.pb
   - peer channel signconfigtx -f org2_update_in_envelope.pb
   - peer channel update -f org2_update_in_envelope.pb -c mychannel -o orderer.example.swarm.com:7050


Host2:
------------
docker exec -it peer0.org2.example-swarm.com /bin/bash
peer channel fetch 0 mychannel.block -o orderer.example.swarm.com:7050 -c mychannel
export CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org2.example.com/msp
peer channel join -b mychannel.block
