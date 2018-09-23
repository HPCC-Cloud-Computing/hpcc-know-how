## Add new Org into an existed channel

* This doc used _the BYFN tutorial - Hyperledger Farbic_ for an example
* Firstly, bring your first network (BYFN) up
    ```sh
    ./byfn.sh -m up -s couchdb
    ```

* Generate Org3 CAs
    ```sh
    cd org3-artifacts
    ../../bin/cryptogen generate --config=./org3-crypto.yaml
    ```

* Generate channel config for Org3 and write out to a .json file
    ```sh
    export FABRIC_CFG_PATH=$PWD && ../../bin/configtxgen -printOrg Org3MSP > ../channel-artifacts/org3.json
    ```

* Copy Orderers CAs to Org3 CAs folder
    ```sh
    cd ../ && cp -r crypto-config/ordererOrganizations org3-artifacts/crypto-config/
    ```

* Prepare CLI env:
    ```sh
    docker exec -it cli bash
    apt update && apt install -y jq
    export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  && export CHANNEL_NAME=mychannel
    ```

* Fetch **the LAST config block** from the orderer and save it as _config_block.pb_ name. In this case, This config block is a config for _Org2 anchor peer update_
    ```sh
    peer channel fetch config config_block.pb -o orderer.example.com:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA
    ```

* Using **configtxlator and jq tools** for traslating _config_block.pb_ to a .json file which is called config.json
    ```sh
    configtxlator proto_decode --input config_block.pb --type common.Block | jq .data.data[0].payload.data.config > config.json
    ```

* Using **jq** tools which has been just installed to append the Org3 configuration definition - _org3.json_ to the channel’s application groups field which was gotten from _config.json_, and name the output – _modified_config.json_
    ```sh
    jq -s '.[0] * {"channel_group":{"groups":{"Application":{"groups": {"Org3MSP":.[1]}}}}}' config.json ./channel-artifacts/org3.json > modified_config.json
    ```

* Now, we will translate _config.json_ and _modified_config.json_ back into protobuf type and then calculate the delta between these two config protobufs, the output file named _org3_update.pb_
    ```sh
    configtxlator proto_encode --input config.json --type common.Config --output config.pb
    configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb
    configtxlator compute_update --channel_id $CHANNEL_NAME --original config.pb --updated modified_config.pb --output org3_update.pb
    ```

* Now we need to wrap org3_update.pb into _an envelope_, this means that we need to write a header for it. 
    ```sh
    - Translate to _org3_update.pb_ to a json file
    configtxlator proto_decode --input org3_update.pb --type common.ConfigUpdate | jq . > org3_update.json
    - Write header for json file
    echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}},"data":{"config_update":'$(cat org3_update.json)'}}}' | jq . > org3_update_in_envelope.json
    - Bring back to protobufs file, the output named _org3_update_in_envelope.pd_
    configtxlator proto_encode --input org3_update_in_envelope.json --type common.Envelope --output org3_update_in_envelope.pb
    ```

* The last one, we will Sign and Submit the config update, we need both signatures from all of Orgs
    * As Org1
    ```sh 
    CORE_PEER_LOCALMSPID="Org1MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    CORE_PEER_ADDRESS=peer0.org1.example.com:7051
    ```
    * And sign for it:
    ```sh
    peer channel signconfigtx -f org3_update_in_envelope.pb
    ```
    * As Org2
    ```sh
    CORE_PEER_LOCALMSPID="Org2MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    CORE_PEER_ADDRESS=peer0.org2.example.com:7051
    ```
    * As Org2, you do not need to run sign cmd, just running update cmd and Org2 Admin signature will be attached to this call
    ```sh
    peer channel update -f org3_update_in_envelope.pb -c $CHANNEL_NAME -o orderer.example.com:7050 --tls --cafile $ORDERER_CA
    ```

* We are done! Now, Org3 can join to the channel. Login as Org3:
    ```sh
    CORE_PEER_LOCALMSPID="Org3MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.example.com/users/Admin@org3.example.com/msp   
    CORE_PEER_ADDRESS=peer0.org3.example.com:7051
    ```

## Create a new channel in the case existing a channel

* In our case, we had three Orgs with their own CAs, now we will create a channel which called _businesschannel_ which consists of Org1 and Org3

* Firstly, we create a configtx.yaml for Org1 and Org3 like the one for Org1 and Org2 before

* Then, we create channel configs for them:
    ```sh
    - The channel name
    CHANNEL_NAME=bussinesschannel
    - The genesis file
    configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
    - The channel config file
    configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/bussinesschannel.tx -channelID $CHANNEL_NAME
    - The anchor Peers
    configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors_bussiness.tx -channelID $CHANNEL_NAME -asOrg Org1MSP
    configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org3MSPanchors_bussiness.tx -channelID $CHANNEL_NAME -asOrg Org3MSP
    ```

* Now, we get into cli and install _jq tool_ if not existing
    ```sh
    docker exec -it cli bash
    apt update && apt install -y jq
    ```

* Export the ORDERER_CA variable
    ```sh
    export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    ```

* To create a new channel, we need to update config for the **system channel** which called _testchainid_ by default. Same as above, firstly, we will fetch the last config block via:
    * Login as Orderer:
    ```sh
    CORE_PEER_LOCALMSPID="OrdererMSP"
    CORE_PEER_TLS_ROOTCERT_FILE=$ORDERER_CA
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/users/Admin@example.com/msp
    ```
    * And then, fetching...
    ```sh
    peer channel fetch config sys_config_block.pb -o orderer.example.com:7050 -c testchainid --tls --cafile $ORDERER_CA
    ```

* Translate block to json format
    ```sh
    curl -X POST --data-binary @sys_config_block.pb "$CONFIGTXLATOR_URL/protolator/decode/common.Block" | jq . > sys_config_block.json
        configtxlator proto_decode --input config_block.pb --type common.Block | jq .data.data[0].payload.data.config > config.json

    configtxlator proto_decode --input sys_config_block.pb --type common.Block | jq . > sys_config_block.json
    ```

* Isolating current config
    ```sh
    jq .data.data[0].payload.data.config sys_config_block.json > sys_config.json
    ```

* Append and add org3.json and then write the output to sys_updated_config.json
    ```sh
    jq -s '.[0] * {"channel_group":{"groups":{"Consortiums":{"groups": {"SampleConsortium": {"groups": {"Org3MSP":.[1]}}}}}}}' sys_config.json ./channel-artifacts/org3.json >& sys_updated_config.json
    ```

* Bring original and updated config back to protobufs type:
    ```sh
    configtxlator proto_encode --input sys_config.json --type common.Config --output sys_config.pb
    configtxlator proto_encode --input sys_updated_config.json --type common.Config --output sys_updated_config.pb
    ```

* Then, calculating the delta between two files
    ```sh
    configtxlator compute_update --channel_id testchainid --original sys_config.pb --updated sys_updated_config.pb --output sys_config_update.pb
    ```
* Decoding config update and Generating config update and wrapping it in an envelope
    ```sh
    configtxlator proto_decode --input sys_config_update.pb --type common.ConfigUpdate | jq . > sys_config_update.json

    echo '{"payload":{"header":{"channel_header":{"channel_id":"testchainid", "type":2}},"data":{"config_update":'$(cat sys_config_update.json)'}}}' | jq . > sys_config_update_in_envelope.json
    ```

* Encoding config update envelope
    ```sh
    configtxlator proto_encode --input sys_config_update_in_envelope.json --type common.Envelope --output sys_config_update_in_envelope.pb
    ```

* As an orderer, sending the config update to its own Orderer node
    ```sh
    peer channel update -f sys_config_update_in_envelope.pb -c testchainid -o orderer.example.com:7050 --tls true --cafile $ORDERER_CA
    ```

* We are done! Now you can create a channel named _bussinesschannel_

    ```sh
    - Creating channel as peer0 in Org3
    peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/bussinesschannel.tx --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA

    - Joining all peers in Org1 & Org3 
    peer channel join -b $CHANNEL_NAME.block

    - Then, updating anchorPeers for Org1 and Org3
    peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/${CORE_PEER_LOCALMSPID}anchors_bussiness.tx --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA >&log.txt
    ```
    * Building chaincode
    ```sh 
    peer chaincode install -n businesscc -v 1.0 -p github.com/chaincode/chaincode_example02/go/

    peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n businesscc -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "OR	('Org1MSP.peer','Org3MSP.peer')"

    peer chaincode invoke -o orderer.example.com:7050  --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n businesscc -c '{"Args":["invoke","a","b","10"]}'

    peer chaincode query -C $CHANNEL_NAME -n businesscc -c '{"Args":["query","a"]}'
    ```