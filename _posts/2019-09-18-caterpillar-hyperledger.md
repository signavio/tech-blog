---
title: "BPX 4 Hyperledger"
description: "How to use the Caterpillar engine to execute business processes on Hyperledger Fabric"
author: Gowtham Mohan, Timotheus Kampik
tags: Blockchain, Hyperledger, Ethereum, Business Process Execution
layout: article
image:
  feature: /2019/web.jpg
  alt: "Web"
---

In this blog post, we show how to execute BPMN 2.0 XML diagrams on Hyperledger Fabric, using the open source tool *Caterpillar* and the *BPMN-Sol* compiler.

## BPX on the Blockchain with Caterpillar
The intersection of blockchain technology and business process management has recently received increased attention by the academic community.
One of the research results is the [Caterpillar](https://github.com/orlenyslp/Caterpillar) engine, a prototype that can compile BPMN 2.0 XML diagrams (with some restrictions) to Solidity smart contracts and execute them on the Ethereum blockchain.
This means, Caterpillar allows business users to specify smart contracts graphically as flowchart-like *BPMN* diagrams; now, you don't need to be a coder to write smart contracts!

![Sample BPMN diagram](../2019/BPMNSampleDiagram.png)
<p align="center">A sample BPMN diagram that can be deployed as smart contract</p>

## Why Hyperledger Fabric matters
By default, Caterpillar integrates with Ethereum (and the Ethereum [Ganache client](https://www.trufflesuite.com/ganache)), which is problematic because Ethereum is typically not considered suitable for Enterprise use cases:

* Ethereum's transaction costs and price fluctuations add unwanted volatility to the business environment.
* The association with *snake oil* schemes and ICOs, as well as the unchecked influence of one individual project lead decrease the willingness of serious enterprises to belief in the long-term sustainability of the ecosystem.

In contrast, the [Hyperledger Fabric](https://www.hyperledger.org/projects/fabric) framework implements a [consortium blockchain](https://hedgetrade.com/blockchain-consortium/) approach: it limits network participation to authenticated parties that are willing to participate in consensus-finding without additional (on-chain) financial incentives.
This makes cryptocurrency features and transaction costs obsolete. 

Also, Hyperledger is backed by well-known industry giants like SAP and IBM, and managed by the Linux Foundation, which provides a comparably high degree of institutional trust.

## Getting it running
The following instructions explain how to execute BPMN 2.0 XML diagrams on Hyperledger Fabric, using the open source tool *Caterpillar*. It is also possible to compile the BMPN 2.0 XML using the [BPMN-Sol compiler](https://github.com/signavio/BPMN-Sol) and deploy to Hyperledger. Both the methods are provided below.


### Prerequisites for this demo

- [Go](https://golang.org/dl/) (version 1.11.1)
- [Docker](https://www.docker.com/) (version 17.06.2-ce or greater)
- [Node](https://nodejs.org/en/) (version 8.9.x or greater)
- [npm](https://www.npmjs.com/) (version 5.6.0)

### Setting the hyperledger network

In this step, we will set up fabric locally using docker containers, install the EVM chain code (EVMCC) and instantiate the EVMCC on our fabric peers. This step uses the Hyperledger [fabric-sample](https://github.com/hyperledger/fabric-samples) repo to deploy fabric locally and the [fabric-chaincode-evm](https://github.com/hyperledger/fabric-chaincode-evm) repo for the EVMCC, Fab3 and Fab3.  This step follows the [fabric-chaincode-evm tutorial](https://github.com/hyperledger/fabric-chaincode-evm/blob/master/examples/EVM_Smart_Contracts.md) closely.

### Clean docker and set GOPATH

This will remove all your docker containers and images!
```
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
docker rmi $(docker images -q) -f
```

Make sure to set GOPATH to your *go* installation:
```
export GOPATH=$HOME/go
```

### Get Fabric Samples and download Fabric images

Clone the [fabric-samples](https://github.com/hyperledger/fabric-samples) repo in your `GOPATH/src/github.com/hyperledger` directory:
```
cd $GOPATH/src/github.com/hyperledger/
git clone https://github.com/hyperledger/fabric-samples.git
```

Checkout `release-1.4`
```
cd fabric-samples
git checkout release-1.4
```

Now run the following command:
```
curl -sSL http://bit.ly/2ysbOFE | bash -s

```

### Mount the EVM chain code and start the network

Clone the `fabric-chaincode-evm` repo in your GOPATH directory.
```
cd $GOPATH/src/github.com/hyperledger/
git clone https://github.com/hyperledger/fabric-chaincode-evm
```

Checkout `release-0.1`
```
cd fabric-chaincode-evm
git checkout release-0.1
```


Now navigate back to your fabric-samples folder.  Here we will use the *first-network* example to launch the network.
```
cd $GOPATH/src/github.com/hyperledger/fabric-samples/first-network
```

Update the `docker-compose-cli.yaml` and include ``fabric-chaincode-evm`` in the list of *volumes*.

```
  cli:
    volumes:
      - ./../../fabric-chaincode-evm:/opt/gopath/src/github.com/hyperledger/fabric-chaincode-evm
```

Generate certificates and bring up the network
```
./byfn.sh generate
./byfn.sh up
```

### Install and Instantiate EVM chain code

Navigate into the cli docker container:

```
docker exec -it cli bash
```

If successful, you should see the following prompt:
```
root@0d78bb69300d:/opt/gopath/src/github.com/hyperledger/fabric/peer#
```

To change which peer is targeted change the following environment variables:
```
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

Clone the Hyperledger EVM chain code project on the Docker VM:

```
git clone https://github.com/hyperledger/fabric-chaincode-evm.git
```

Next, install the EVM chain code on all the peers:
```
peer chaincode install -n evmcc -l golang -v 0 -p github.com/hyperledger/fabric-chaincode-evm/evmcc
```

Instantiate the chain code:

```
peer chaincode instantiate -n evmcc -v 0 -C mychannel -c '{"Args":[]}' -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

Great.  You can exit out of the cli container and return to your terminal.
```
exit
```

You are now ready to setup Fab3.


### Set up Fab3

**Open a new terminal window** and execute the following to set certain environment variables required for setting up Fab3.

```
export FABPROXY_CONFIG=${GOPATH}/src/github.com/hyperledger/fabric-chaincode-evm/examples/first-network-sdk-config.yaml # Path to a compatible Fabric SDK Go config file
export FABPROXY_USER=User1 # User identity being used for the proxy (Matches the users names in the crypto-config directory specified in the config)
export FABPROXY_ORG=Org1  # Organization of the specified user
export FABPROXY_CHANNEL=mychannel # Channel to be used for the transactions
export FABPROXY_CCID=evmcc # ID of the EVM Chaincode deployed in your fabric network
export PORT=8545 # Port the proxy will listen on. If not provided default is 8545.
```

Navigate to the `fabric-chaincode-evm` cloned repo:
```
cd $GOPATH/src/github.com/hyperledger/fabric-chaincode-evm/
```
Run the following to build the fab proxy:
```
go build -o fab3 ./fabproxy/cmd
```
You can then run the proxy:
```
./fab3
```

This will start Fab3 at `http://localhost:8545`.
You should see output like the following:
```
{"level":"info","ts":1550530404.3546276,"logger":"fab3","caller":"cmd/main.go:143","msg":"Starting Fab3","port":8545}
```
Now we have successfully set up the local Hyperledger network. The next step is to start the Caterpillar engine. 

### Method 1:Caterpillar engine
### Usage

Open a new terminal and run the Caterpillar docker image with ``docker run --rm -it -p 3200:3200 -p 3000:3000 -p 8090:8090 gowthammohan/caterpillarmodified``.
Note that we provide and use a customized version of the Caterpillar v1 engine.
After the caterpillar engine has started, the execution panel, which lets us to interact with the engine, is now available at ``http://localhost:3200``.


Go to the execution panel at ``http://localhost:3200``.
Here you can either upload a BPMN diagram or model a diagram using the modeler provided.
It is important for the BPMN diagram to have proper element documentation as specified in [Caterpillar v1 documentation](http://ceur-ws.org/Vol-1920/BPM_2017_paper_199.pdf).
To use the sample BPMN diagram given above use this [SampleBPMN](https://github.com/signavio/Caterpillar/blob/fabric/SampleBPMN.bpmn).
Save this BPMN 2.0 XML in a file and upload it to the execution panel. Once the diagram is loaded, specify these global variables in the element documentation.:

``` 
uint userID; 
bool applicantEligible = false; 
``` 
On click of **Save** the engine starts compiling the diagram to a smart contract (chain code) and the contract details are logged on the console.


### Method 2: Using the BPMN-Sol compiler
### Usage

Clone this repository [BPMN-Sol compiler](https://github.com/signavio/BPMN-Sol) to your machine and `cd` into the project directory and run `npm install` to install the dependencies. In the `index.ts` file there is a sample xml object which can be compiled to smart contract.
```
const xml = {
  bpmn:
    '<?xml version="1.0" encoding="UTF-8"?>\n<bpmn:definitions xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:bpmn="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:dc="http://www.omg.org/spec/DD/20100524/DC" xmlns:di="http://www.omg.org/spec/DD/20100524/DI" id="Definitions_1" targetNamespace="http://bpmn.io/schema/bpmn">\n  <bpmn:process id="Process_0" name="Process_0" isExecutable="false">\n    <bpmn:startEvent id="StartEvent_1">\n      <bpmn:outgoing>SequenceFlow_005ctt4</bpmn:outgoing>\n    </bpmn:startEvent>\n    <bpmn:task id="Task_1hm78g6" name="a">\n      <bpmn:incoming>SequenceFlow_005ctt4</bpmn:incoming>\n      <bpmn:outgoing>SequenceFlow_06osrd7</bpmn:outgoing>\n    </bpmn:task>\n    <bpmn:sequenceFlow id="SequenceFlow_005ctt4" sourceRef="StartEvent_1" targetRef="Task_1hm78g6" />\n    <bpmn:endEvent id="EndEvent_1rfz6w8">\n      <bpmn:incoming>SequenceFlow_06osrd7</bpmn:incoming>\n    </bpmn:endEvent>\n    <bpmn:sequenceFlow id="SequenceFlow_06osrd7" sourceRef="Task_1hm78g6" targetRef="EndEvent_1rfz6w8" />\n  </bpmn:process>\n  <bpmndi:BPMNDiagram id="BPMNDiagram_1">\n    <bpmndi:BPMNPlane id="BPMNPlane_1" bpmnElement="Process_0">\n      <bpmndi:BPMNShape id="_BPMNShape_StartEvent_2" bpmnElement="StartEvent_1">\n        <dc:Bounds x="173" y="102" width="36" height="36" />\n      </bpmndi:BPMNShape>\n      <bpmndi:BPMNShape id="Task_1hm78g6_di" bpmnElement="Task_1hm78g6">\n        <dc:Bounds x="275" y="80" width="100" height="80" />\n      </bpmndi:BPMNShape>\n      <bpmndi:BPMNEdge id="SequenceFlow_005ctt4_di" bpmnElement="SequenceFlow_005ctt4">\n        <di:waypoint xsi:type="dc:Point" x="209" y="120" />\n        <di:waypoint xsi:type="dc:Point" x="275" y="120" />\n        <bpmndi:BPMNLabel>\n          <dc:Bounds x="242" y="98" width="0" height="13" />\n        </bpmndi:BPMNLabel>\n      </bpmndi:BPMNEdge>\n      <bpmndi:BPMNShape id="EndEvent_1rfz6w8_di" bpmnElement="EndEvent_1rfz6w8">\n        <dc:Bounds x="427" y="102" width="36" height="36" />\n        <bpmndi:BPMNLabel>\n          <dc:Bounds x="445" y="141" width="0" height="13" />\n        </bpmndi:BPMNLabel>\n      </bpmndi:BPMNShape>\n      <bpmndi:BPMNEdge id="SequenceFlow_06osrd7_di" bpmnElement="SequenceFlow_06osrd7">\n        <di:waypoint xsi:type="dc:Point" x="375" y="120" />\n        <di:waypoint xsi:type="dc:Point" x="427" y="120" />\n        <bpmndi:BPMNLabel>\n          <dc:Bounds x="401" y="98" width="0" height="13" />\n        </bpmndi:BPMNLabel>\n      </bpmndi:BPMNEdge>\n    </bpmndi:BPMNPlane>\n  </bpmndi:BPMNDiagram>\n</bpmn:definitions>\n',
  name: "Process_0"
};
```
You can also add your xml file however it has to be in the above format.(An xml object with the xml value as the string and the name of the contract). Now add the following code in index.ts to log the contract details in the terminal.
```
const contract = compile(xml).then(contract => {
   console.log(contract);
})
```
Run `npm run build` and `npm start`.
The compile function returns the Solidity code, Bytecode and ABI value which are essential for deploying the smart contract to hyperledger.


### Deploying the smart contract.

We will install the web3 dependency and use this library to deploy the smart contract.

To install the same version of `web3`, open a new terminal and run:

```
npm install web3@0.20.2
```

Now, we'll enter node console to set up our ``web3``.

```
node
```

Assign Web3 library and use *fab3* running in the previous terminal as the provider.

```
Web3 = require('web3')
web3 = new Web3(new Web3.providers.HttpProvider('http://localhost:8545'))
```

Check to see your account information:

```
web3.eth.accounts
```

You should see something like this which is similar to an Ethereum account:
```
[ '0x2c045d4565e31cef1f6cd7368c3436a79f1cea4f' ]
```
Assign this account as ``defaultAccount``:

```web3.eth.defaultAccount = web3.eth.accounts[0]```

From the values obtained through either method 1 or method 2 assign the Bytecode and ABI values.

```ABI = ABI value```

Next, assign the long evm complied byte code:  

ByteCode = `Bytecode value`


Assign the contract with web3 using the contract's ABI.
```
Contract = web3.eth.contract(ABI)
```

Next, deploy the contract using the contract byte code:

```
deployedContract = Contract.new([], { data: ByteCode })
```

You can get the contract address by using the transaction hash of the deployed contract:

```
web3.eth.getTransactionReceipt(deployedContract.transactionHash)
```

Now, the BPMN 2.0 XML diagram has been successfuly deployed to the Hyperledger network.

## Conclusion
As we have shown in this blog post, executing BPMN diagrams on Hyperledger Fabric is relatively straight-forward after making minor adjustments to Caterpillar.
Still, we think that more work is necessary to allow for the business user-friendly specification and deployment of chain code:

* **Test-driven chain code specification**  
  Considering the software development best practice of *test driven* development, process modelers should be able to specify test cases that describe the target behavior directly in their modeling environment.
* **Continuous integration**
  The process of chain code modeling, testing and deployment should be supported by continuous integration pipelines.


<a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@clintadair?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Clint Adair"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-2px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M10 9V0h12v9H10zm12 5h10v18H0V14h10v9h12v-9z"></path></svg></span><span style="display:inline-block;padding:2px 3px">Clint Adair</span></a>
