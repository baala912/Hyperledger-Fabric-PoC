# Create and Deploy a Blockchain Network using Hyperledger Fabric SDK Java

> **Note:** This code pattern builds a Hyperledger Fabric 1.4 network and uses Hyperledger Fabric SDK java 1.4

## Flow

   ![](images/architecture.png)

1. Generate the artifacts using cryptogen and configtx for peers and channel in network. Currently these are already generated and provided in the code repository to use as-is.
2. Build the network using docker-compose and the generated artifacts.
3. Use Hyperledger Fabric Java SDK APIs to work with and manage the network.
    * Create and initialize the channel
    * Install and instantiate the chaincode
    * Register and enroll the users
    * Perform invoke and query to test the network


## Included Components

* [Hyperledger Fabric](https://hyperledger-fabric.readthedocs.io/): Hyperledger Fabric is a platform for distributed ledger solutions underpinned by a modular architecture delivering high degrees of confidentiality, resiliency, flexibility and scalability.

* [Docker](https://www.docker.com/): Docker is an open platform for developers and sysadmins to build, ship, and run distributed applications.

* [Hyperledger Fabric Java SDK](https://github.com/hyperledger/fabric-sdk-java)

## Featured Technologies

* [Blockchain](https://en.wikipedia.org/wiki/Blockchain): A blockchain is a digitized, decentralized, public ledger of all transactions in a network.

* [Java](https://en.wikipedia.org/wiki/Java_(programming_language)): Java is a general-purpose computer-programming language that is concurrent, class-based and object-oriented.


## Pre-requisites

* [Docker](https://www.docker.com/get-started) 
* [Docker Compose](https://docs.docker.com/compose/overview/)
* [Git Client](https://git-scm.com/downloads) - needed for clone commands
* [Maven](https://maven.apache.org/download.cgi) - needed to build the client. Maven is a build automation tool used primarily for Java projects. Maven addresses two aspects of building software: first, it describes how software is built, and second, it describes its dependencies.

## Steps

Follow these steps to setup and run this code pattern.

1. [Setup the Blockchain Network](#1-setup-the-blockchain-network)
2. [Build the client based on Fabric Java SDK](#2-build-the-client-based-on-fabric-java-sdk)
3. [Create and Initialize the channel](#3-create-and-initialize-the-channel)
4. [Deploy and Instantiate the chaincode](#4-deploy-and-instantiate-the-chaincode)
5. [Register and enroll users](#5-register-and-enroll-users)
6. [Perform Invoke and Query on network](#6-perform-invoke-and-query-on-network)

### 1. Setup the Blockchain Network

[Clone this repo](https://github.com/IBM/blockchain-application-using-fabric-java-sdk) using the following command.

```
$ git clone https://github.com/IBM/blockchain-application-using-fabric-java-sdk
```

To build the blockchain network, the first step is to generate artifacts for peers and channels using cryptogen and configtx. The utilities used and steps to generate artifacts are explained [here](https://hyperledger-fabric.readthedocs.io/en/release-1.4/build_network.html). In this pattern all required artifacts for the peers and channel of the network are already generated and provided to use as-is. Artifacts can be located at:

   ```
   network_resources/crypto-config
   network_resources/config
   ````

The automated scripts to build the network are provided under `network` directory. The `network/docker-compose.yaml` file defines the blockchain network topology. This pattern provisions a Hyperledger Fabric 1.4.1 network consisting of two organizations, each maintaining two peer node, two certificate authorities for each organization and a solo ordering service. Need to run the script as follows to build the network.

> **Note:** Please clean up the old docker images (if any) from your environment otherwise you may get errors while setting up network.

   ```
   cd network
   chmod +x build.sh
   ./build.sh
   ```

To stop the running network, run the following script.

   ```
   cd network
   chmod +x stop.sh
   ./stop.sh
   ```

To delete the network completely, following script need to execute.

   ```
   cd network
   chmod +x teardown.sh
   ./teardown.sh
   ```

### 2. Build the client based on Fabric Java SDK

The previous step creates all required docker images with the appropriate configuration.

**Java Client**
* The java client sources are present in the folder `java` of the repo.
* Check your environment before executing the next step. Make sure, you are able to run `mvn` commands properly.
   > If `mvn` commands fails, please refer to [Pre-requisites](#pre-requisites) to install maven.


To work with the deployed network using Hyperledger Fabric SDK java 1.4.1, perform the following steps.

* Open a command terminal and navigate to the `java` directory in the repo. Run the command `mvn install`.

   ```
   cd ../java
   mvn install
   ```

* A jar file `blockchain-java-sdk-0.0.1-SNAPSHOT-jar-with-dependencies.jar` is built and can be found under the `target` folder. This jar can be renamed to `blockchain-client.jar` to keep the name short.

   ```
   cd target
   cp blockchain-java-sdk-0.0.1-SNAPSHOT-jar-with-dependencies.jar blockchain-client.jar
   ```

* Copy this built jar into `network_resources` directory. This is required as the java code can access required artifacts during execution.

   ```
   cp blockchain-client.jar ../../network_resources
   ```

### 3. Create and Initialize the channel

In this code pattern, we create one channel `mychannel` which is joined by all four peers. The java source code can be seen at  `src/main/java/org/example/network/CreateChannel.java`. To create and initialize the channel, run the following command.

   ```
   cd ../../network_resources
   java -cp blockchain-client.jar org.example.network.CreateChannel
   ```

Output:

   ```Apr 20, 2018 5:11:42 PM org.example.util.Util deleteDirectory
      INFO: Deleting - users
      Apr 20, 2018 5:11:45 PM org.example.network.CreateChannel main
      INFO: Channel created mychannel
      Apr 20, 2018 5:11:45 PM org.example.network.CreateChannel main
      INFO: peer0.org1.example.com at grpc://localhost:7051
      Apr 20, 2018 5:11:45 PM org.example.network.CreateChannel main
      INFO: peer1.org1.example.com at grpc://localhost:7056
      Apr 20, 2018 5:11:45 PM org.example.network.CreateChannel main
      INFO: peer0.org2.example.com at grpc://localhost:8051
      Apr 20, 2018 5:11:45 PM org.example.network.CreateChannel main
      INFO: peer1.org2.example.com at grpc://localhost:8056
   ```

### 4. Deploy and Instantiate the chaincode

This code pattern uses a sample chaincode `fabcar` to demo the usage of Hyperledger Fabric SDK Java APIs. To deploy and instantiate the chaincode, execute the following command.

   ```
   java -cp blockchain-client.jar org.example.network.DeployInstantiateChaincode
   ```

   Output:

   ```
      log4j:WARN No appenders could be found for logger (org.hyperledger.fabric.sdk.helper.Config).
      log4j:WARN Please initialize the log4j system properly.
      log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
      WARNING: An illegal reflective access operation has occurred
      WARNING: Illegal reflective access by org.bouncycastle.jcajce.provider.drbg.DRBG (file:/Users/bala/Desktop/workwolf/blockchain-application-using-fabric-java-sdk/network_resources/blockchain-client.jar) to constructor sun.security.provider.Sun()
      WARNING: Please consider reporting this to the maintainers of org.bouncycastle.jcajce.provider.drbg.DRBG
      WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
      WARNING: All illegal access operations will be denied in a future release
      Aug. 27, 2020 3:59:55 P.M. org.example.client.FabricClient deployChainCode
      INFO: Deploying chaincode usercredentials using Fabric client Org1MSP admin
      Aug. 27, 2020 3:59:55 P.M. org.example.network.DeployInstantiateChaincode main
      INFO: usercredentials- Chain code deployment SUCCESS
      Aug. 27, 2020 3:59:55 P.M. org.example.network.DeployInstantiateChaincode main
      INFO: usercredentials- Chain code deployment SUCCESS
      Aug. 27, 2020 3:59:55 P.M. org.example.client.FabricClient deployChainCode
      INFO: Deploying chaincode usercredentials using Fabric client Org2MSP admin
      Aug. 27, 2020 3:59:55 P.M. org.example.network.DeployInstantiateChaincode main
      INFO: usercredentials- Chain code deployment SUCCESS
      Aug. 27, 2020 3:59:55 P.M. org.example.network.DeployInstantiateChaincode main
      INFO: usercredentials- Chain code deployment SUCCESS
      Aug. 27, 2020 3:59:55 P.M. org.example.client.ChannelClient instantiateChainCode
      INFO: Instantiate proposal request usercredentials on channel mychannel with Fabric client Org2MSP admin
      Aug. 27, 2020 3:59:55 P.M. org.example.client.ChannelClient instantiateChainCode
      INFO: Instantiating Chaincode ID usercredentials on channel mychannel
      Aug. 27, 2020 4:00:34 P.M. org.example.client.ChannelClient instantiateChainCode
      INFO: Chaincode usercredentials on channel mychannel instantiation java.util.concurrent.CompletableFuture@9cd25ff[Not completed]
      Aug. 27, 2020 4:00:34 P.M. org.example.network.DeployInstantiateChaincode main
      INFO: usercredentials- Chain code instantiation SUCCESS
      Aug. 27, 2020 4:00:34 P.M. org.example.network.DeployInstantiateChaincode main
      INFO: usercredentials- Chain code instantiation SUCCESS
      Aug. 27, 2020 4:00:34 P.M. org.example.network.DeployInstantiateChaincode main
      INFO: usercredentials- Chain code instantiation SUCCESS
      Aug. 27, 2020 4:00:34 P.M. org.example.network.DeployInstantiateChaincode main
      INFO: usercredentials- Chain code instantiation SUCCESS
   ```

   > **Note:** The chaincode fabcar.go was taken from the fabric samples available at - https://github.com/hyperledger/fabric-samples/tree/release-1.4/chaincode/fabcar/go.

### 5. Register and enroll users

A new user can be registered and enrolled to an MSP. Execute the below command to register a new user and enroll to Org1MSP.

   ```
   java -cp blockchain-client.jar org.example.user.RegisterEnrollUser
   ```

   Output:

   ```Apr 23, 2018 10:26:34 AM org.example.util.Util deleteDirectory
      INFO: Deleting - users
      log4j:WARN No appenders could be found for logger (org.hyperledger.fabric.sdk.helper.Config).
      log4j:WARN Please initialize the log4j system properly.
      log4j:WARN See https://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
      Apr 23, 2018 10:26:35 AM org.example.client.CAClient enrollAdminUser
      INFO: CA -http://localhost:7054 Enrolled Admin.
      Apr 23, 2018 10:26:35 AM org.example.client.CAClient registerUser
      INFO: CA -http://localhost:7054 Registered User - user1524459395783
      Apr 23, 2018 10:26:36 AM org.example.client.CAClient enrollUser
      INFO: CA -http://localhost:7054 Enrolled User - user1524459395783
   ```

### 6. Perform Invoke and Query on network

Blockchain network has been setup completely and is ready to use. Now we can test the network by performing invoke and query on the network. The `fabcar` chaincode allows us to create a new asset which is a car. For test purpose, invoke operation is performed to create a new asset in the network and query operation is performed to list the asset of the network. Perform the following steps to check the same.

   ```
   java -cp blockchain-client.jar org.example.chaincode.invocation.InvokeChaincode
   ```

   Output:

   ```Aug. 27, 2020 4:01:21 P.M. org.example.util.Util deleteDirectory
      INFO: Deleting - users
      log4j:WARN No appenders could be found for logger (org.hyperledger.fabric.sdk.helper.Config).
      log4j:WARN Please initialize the log4j system properly.
      log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
      WARNING: An illegal reflective access operation has occurred
      WARNING: Illegal reflective access by org.bouncycastle.jcajce.provider.drbg.DRBG (file:/Users/bala/Desktop/workwolf/blockchain-application-using-fabric-java-sdk/network_resources/blockchain-client.jar) to constructor sun.security.provider.Sun()
      WARNING: Please consider reporting this to the maintainers of org.bouncycastle.jcajce.provider.drbg.DRBG
      WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
      WARNING: All illegal access operations will be denied in a future release
      Aug. 27, 2020 4:01:22 P.M. org.example.client.CAClient enrollAdminUser
      INFO: CA -http://localhost:7054 Enrolled Admin.
      Aug. 27, 2020 4:01:22 P.M. org.example.client.ChannelClient sendTransactionProposal
      INFO: Sending transaction proposal on channel mychannel
      Aug. 27, 2020 4:01:22 P.M. org.example.client.ChannelClient sendTransactionProposal
      INFO: Transaction proposal on channel mychannel  SUCCESS with transaction id:46a42e0448a8ada971e473cc0b868ebdc124b60d9ef05be2d62c733be836ddf4
      Aug. 27, 2020 4:01:22 P.M. org.example.client.ChannelClient sendTransactionProposal
      INFO: 
      Aug. 27, 2020 4:01:22 P.M. org.example.client.ChannelClient sendTransactionProposal
      INFO: java.util.concurrent.CompletableFuture@7c8326a4[Not completed]
      Aug. 27, 2020 4:01:22 P.M. org.example.chaincode.invocation.InvokeChaincode main
      INFO: Invoked createUser on usercredentials. Status - SUCCESS
  ```

   ```
   java -cp blockchain-client.jar org.example.chaincode.invocation.QueryChaincode
   ```

   Output:

   ```
    Aug. 27, 2020 4:02:22 P.M. org.example.util.Util deleteDirectory
    INFO: Deleting - admin.ser
    Aug. 27, 2020 4:02:22 P.M. org.example.util.Util deleteDirectory
    INFO: Deleting - org1
    Aug. 27, 2020 4:02:22 P.M. org.example.util.Util deleteDirectory
    INFO: Deleting - users
    log4j:WARN No appenders could be found for logger (org.hyperledger.fabric.sdk.helper.Config).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    WARNING: An illegal reflective access operation has occurred
    WARNING: Illegal reflective access by org.bouncycastle.jcajce.provider.drbg.DRBG (file:/Users/bala/Desktop/workwolf/blockchain-application-using-fabric-java-sdk/network_resources/blockchain-client.jar) to constructor sun.security.provider.Sun()
    WARNING: Please consider reporting this to the maintainers of org.bouncycastle.jcajce.provider.drbg.DRBG
    WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
    WARNING: All illegal access operations will be denied in a future release
    Aug. 27, 2020 4:02:23 P.M. org.example.client.CAClient enrollAdminUser
    INFO: CA -http://localhost:7054 Enrolled Admin.
    Aug. 27, 2020 4:02:23 P.M. org.example.chaincode.invocation.QueryChaincode main
    INFO: Querying for a user - USER1
    Aug. 27, 2020 4:02:23 P.M. org.example.client.ChannelClient queryByChainCode
    INFO: Querying queryUser on channel mychannel
    Aug. 27, 2020 4:02:23 P.M. org.example.chaincode.invocation.QueryChaincode main
    INFO: {"ssnAccess":["USER2"]}
   ```

## Troubleshooting

[See DEBUGGING.md.](DEBUGGING.md)

