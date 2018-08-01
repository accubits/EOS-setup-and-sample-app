# EOS setup and sample app
##  **SETUP**
 **Prerequisite**
- Git
- Ubuntu 16 or 18 LTS  (Ubuntu 16.10 recommended)

**System Requirements (all platforms)**
- 7GB RAM free required
- 20GB Disk free required

**Overview**
EOS comes with mainly three modules.
- nodeos  - the core EOSIO node daemon
- cleos - command line interface to interact with the blockchain and to manage wallets 
- keosd - component that securely stores EOSIO keys in wallets. 

![N|Solid](https://files.readme.io/8f31cfd-Basic-EOSIO-System-Architecture.png)

 **Initial installation**
 
```sh
$ git clone https://github.com/eosio/eos --recursive
$ cd eos
```
It will download all the EOS code and its sub-modules

**Build the code**
EOS code can built with multiple ways based on the platform and the desired use.
- Autobuild Script
- Docker compose
- Manual build
- Install executables

Autobuild script is the one suitable for the developers and we are setting up that model.
```sh
$ ./eosio_build.sh 
```
`This will take a  couple of hours depending on the spec of machine`

**Build validation**
Test the build operation’s status(Validation Test) in eos/build directory using the following testing commands.
```sh
#Start mongo
$ ~/opt/mongodb/bin/mongod -f ~/opt/mongodb/mongod.conf &
$ cd build
$ make test
```
**Install the executables**
```sh
$ sudo make install
```
#### **Single node network**
After building the project, you can start the single node by running nodeos
```sh
$ cd ~/eos/build/programs/nodeos
$ ./nodeos -e -p eosio --plugin eosio::wallet_api_plugin --plugin eosio::chain_api_plugin --plugin eosio::history_api_plugin 
```
-e :-    Enable block production
-p  :-   ID of producer controller by the node
--plugin :- To enable multiple plugins

Nodeos uses a configuration folder.The config file ~/.local/share/eosio/nodeos/config can be found in this location.If config.ini is not found a default config.ini is created.

*Issues may arise:-*  
*3190000 block_log_exception: Block log exception*
*Block log was not setup properly with genesis information.*
*Solution* 
*Clear this folder for the issue :-*
*rm -rf ~/.local/share/eosio/nodeos/data*
*Or  ./nodeos --replay-blockchain --hard-replay-blockchain*

#### **Wallet Creation**
Open a new terminal
```sh
$ cd ~/eos/build/programs/cleos 
$ ./cleos wallet create  -n ``<your wallet name> ``
$ ./cleos create key
```
Sample Response
``Private key: 5JmJyTf9cYemDctq3a28UM2YDmr3eDStsHSXv1VqgUbCaHTuCbn``
   ``Public key: EOS8akHPngGi1nfvR55WqiGrTeCtsSLZRvAGT5LBRdrXDmoqQpSXw``

**Import  Private key**
```sh
./cleos wallet import ``<privatekey>``
```
# **Introduction about EOSIO Smart Contract**

   > An EOSIO Smart Contract is software registered on the blockchain and executed on EOSIO nodes, that implements the semantics of a "contract" whose ledger of action requests are being stored on the blockchain.
   > The Smart Contract defines the interface (actions, parameters, data structures) and the code that implements the interface. 
   >  The code is compiled into a canonical bytecode format that nodes can retrieve and execute. 
   >  The blockchain stores the transactions of the contract. 
   >  Every eosio smart contract belongs to a single eosio account.
   > Performing an action to a contract requires the authorization of at least one account. 
   > An account can be made of a single, or many, individuals set up in a permission based configuration. 
Smart contracts can only be run by a single account, and a single account can only have a single smart contract.
   > Note :-
   Best practice - Use the same (lowercase) name for both the account and contract.

## **The eosio.token contract explanation**
   > Allows the creation/transfer of tokens.
   > Every token contract is owned by an owner/issuer account.
   
## **Steps for Contract Compilation & Deployment**
 1)Account Creation 
    
    `   ./cleos create account eosio eosio.token <OWNER-KEY> <ACTIVE-KEY>   `
    
    
 Usage
              ` ./cleos create account [OPTIONS] creator name OwnerKey ActiveKey `

 ##### Positionals:
 > creator TEXT                The name of the account creating the new account 
 
 > name TEXT                   The name of the new account
 
 > OwnerKey TEXT               The owner public key for the new account
 
 > ActiveKey TEXT              The active public key for the new account
                     
Sample Input & Response
    
           ` ./cleos create account eosio eosio.token  EOS8akHPngGi1nfvR55WqiGrTeCtsSLZRvAGT5LBRdrXDmoqQpSXw EOS5Bi2rhVvVH7JLUwHabEkKkZg4LFB8ifehxoMx8hF6TeZGMWUFQ `
           
           ` executed transaction: a116a8b0b077132adea1619de2afe1066dbcf68610be0e42fae0c0f7c0240817  200 bytes  45966 us
         eosio <= eosio::newaccount            {"creator":"eosio","name":"eosio.token","owner":{"threshold":1,"keys":[{"key":"EOS8akHPngGi1nfvR55Wq...
     2018-08-01T06:01:01.269 thread-0   main.cpp:391                  print_result   warning: transaction executed locally, but may not be confirmed by the network yet `

2) Contract Compilation


        (i) ` cd eos/contract/eosio.token `
        (ii) ` eosiocpp -o eosio.token.wast eosio.token.cpp `
        
 #######Note about issues
        
              ` eosiocpp:command not found `
              
  Solution
              
              ` sh cd eos/build `
              ` sh sudo make install `
              
 This command will install and shows the location of installed binaries.
              
              ` sh -- Up-to-date: /usr/local/eosio/usr/share/eosio/contractsdk/lib/identity_interface.bc
                 -- Up-to-date: /usr/local/eosio/bin/nodeos
                 -- Up-to-date: /usr/local/eosio/var/log/eosio
                 -- Up-to-date: /usr/local/eosio/var/lib/eosio
                 -- Up-to-date: /usr/local/eosio/bin/cleos
                 -- Up-to-date: /usr/local/eosio/bin/keosd
                 -- Up-to-date: /usr/local/eosio/bin/eosio-launcher
                 -- Up-to-date: /usr/local/eosio/bin/eosio-abigen
                 -- Up-to-date: /usr/local/eosio/bin/eosiocpp `
            
 Add these location to your path as follows.
            
              ` export PATH=/usr/local/eosio/bins:$PATH
                 which eosiocpp
                            /usr/bin/eosio/bin/eosiocpp```
Then your compilation will create the eosio.token.wasm and eosio.token.abi

3) Contract deployment

                 ` sh ./cleos set contract {account} {path_to_contract_folder} {path_to_wast_file} {path_to_abi_file} `
   ##### Positional Parameters :-
   > account TEXT - The account to publish a contract for
   
   > wast-file TEXT - The file containing the contract WAST or WASM
   
   > abi-file TEXT - The ABI for the contract
            
             
    Sample Input :-
                  `  ./cleos set contract eosio.token  ../../../contracts/eosio.token `
                  
    Response
                 
                 ` Reading WAST/WASM from ../../../contracts/eosio.token/eosio.token.wasm...
                   Using already assembled WASM...
                   Publishing contract...
                   executed transaction: 2400492e9729cd94d8c89703f1cb343643f18def9f605ad0985ef7422ff4bd74  8112 bytes  127802 us
       eosio <= eosio::setcode               {"account":"eosio.token","vmtype":0,"vmversion":0,"code":"0061736d01000000017e1560037f7e7f0060057f7e...
       eosio <= eosio::setabi                {"account":"eosio.token","abi":"0e656f73696f3a3a6162692f312e30010c6163636f756e745f6e616d65046e616d65...
                     2018-08-01T11:19:39.585 thread-0   main.cpp:391                  print_result   warning: transaction executed locally, but may not be confirmed by the network yet```
      
  4) To verify the contract deployment
            ` ./cleos get code eosio.token`
            
   Response :-
            
            ` code hash: 9210637cb6abf6ac8e887cb43c75ca3f86713c9b3b668b0ec63b6e39183b54fd `
     
     
                 
#### **EOSIO Token SmartContract Interaction**
###### ***Coming Soon***
#### **EOSIO Token SmartContract Explanation**
###### ***Coming Soon***

References: [EOSIO](https://developers.eos.io/eosio-nodeos/docs)

Contributors:
- [Crissi](https://github.com/crissiaccubits)
- [Goutham](https://github.com/goutham-ab)
