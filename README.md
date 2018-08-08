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
1) Create Function

 Notes:-
 Push Action  :- Push a transaction with a single action
            
 Positionals
            
  >contract Type: Text - The account providing the contract to execute
  
  >action Type: Text - The action to execute on the contract
  
  >data Type: Text - The arguments to the contract
            
 ` ./cleos push action eosio.token create '{"issuer":"eosio.token","maximum_supply":"1000000.0000 TKN","can_freeze":"0","can_recall":"0","can_whitelist":"0"}' -p eosio.token`
Issues :-  `Error 3080006: Transaction took too long`

Solution:- append -x after the set contract command

  > -x,--expiration :-  set the time in seconds before a transaction expires, defaults to 30s
           
           
Updated Command :-
           `./cleos push action eosio.token create '{"issuer":"eosio.token","maximum_supply":"1000000.0000 TKN","can_freeze":"0","can_recall":"0","can_whitelist":"0"}' -p eosio.token -x 600`
           
Response:-
       `executed transaction: 8aafe7154c42bfe600a33a6b0ce088d01c846a658e0872264c59ae455bbd659c  120 bytes  917 us`
        `#   eosio.token <= eosio.token::create          {"issuer":"eosio.token","maximum_supply":"1000000.0000 TKN"}
2018-08-02T05:02:51.930 thread-0   main.cpp:391                  print_result   warning: transaction executed locally, but may    not be confirmed by the network yet`

2) Token Issue Function

        ` ./cleos push action eosio.token issue '{"to":"eosio.token","quantity":"1000.0000 TKN","memo":""}' -p eosio.token`
        
   Response :-
   
       `  executed transaction: 389b45416b54b4ec6a5f5b1135f276f0801486835f914fc14a8aa68c5efd605e  120 bytes  882 us
          #   eosio.token <= eosio.token::issue           {"to":"eosio.token","quantity":"1000.0000 TKN","memo":""}
          warning: transaction executed locally, but may not be confirmed by the network yet    ] `
          
          
3) Initial Balance Verification

Notes :-

get table :-  Retrieves the contents of a db table

Positionals :-

>contract TEXT - The contract who owns the table

>scope TEXT - The scope within the contract in which the table is found 

>table TEXT - The name of the table as specified by the contract abi


         ` ./cleos get table eosio.token eosio.token accounts`
         
Response :-
          `{
                      "rows": [{
                                  "balance": "1000.0000 TKN"
                                  }
                        ],
                      "more": false
                 }`
                 

4) Token Transfer

Transfer tokens from “eosio.token” to “eosio”
        
         ` ./cleos push action eosio.token transfer '{"from":"eosio.token","to":"eosio","quantity":"20.0000 TKN","memo":"my first transfer"}' -p eosio.token`
         
Response :-

         `  executed transaction: 6b74bfaff1e613c2185fcc45c2487c5fe2a0abeb0f5243779adb9f2881cc46f0  144 bytes  1336 us
           #   eosio.token <= eosio.token::transfer        {"from":"eosio.token","to":"eosio","quantity":"20.0000 TKN","memo":"my first transfer"}
           #         eosio <= eosio.token::transfer        {"from":"eosio.token","to":"eosio","quantity":"20.0000 TKN","memo":"my first transfer"}
          warning: transaction executed locally, but may not be confirmed by the network yet    ] `

5) Balance checks in two accounts

a)Account 1
      `./cleos get table eosio.token eosio accounts`
      
   Response :-
           
           `{   "rows": [{
                           "balance": "20.0000 TKN"
                          }
                        ],
                              more": false
                       },
             }`
             
 b)Account 2
         ` ./cleos get table eosio.token eosio.token accounts`
         
   Response :-
   
         `{   "rows": [{
                           "balance": "980.0000 TKN"
                          }
                        ],
                              more": false
                       },
             }'
         

      
#### **EOSIO Token SmartContract Explanation**
Contract EOSIO.token

1)  .hpp file defines the contract class,actions,tables.// HPP is a file extension for a header file file format.
2)  .cpp file implements the action logic.

.hpp File Explanation:-

 Stage 1 :-  	 	 	
                      
                     #pragma once
                     #include <eosiolib/asset.hpp>        ---> Defines API for mananging Assests
                     #include <eosiolib/eosio.hpp>        --->  Defines print,contract  
                     #include <string>

                     namespace eosiosystem {
                        class system_contract;
                     }

                      namespace eosio {

                            using std::string;

                           }    //namespace eosio
                           
Stage 2 :-
//constructor and actions are defined as public members

//constructor takes an account name (which will be the account that the contract is deployed on, aka eosio.token) and sets it to the contract variable. 

//The  class is inheriting from the ‘eosio::contract’.

                             
                             class token : public contract {           
                   
                                     public:
                                          //constructor
                                          token( account_name self ):contract(self){}
                                          //action
                                          void create( account_name issuer,asset maximum_supply);
                                          //action
                                          void issue( account_name to, asset quantity, string memo );
                                          //action
                                          void transfer( account_name from,
                                                         account_name to,
                                                         asset        quantity,
                                                         string       memo );
    
   
                                         inline asset get_supply( symbol_name sym )const;
       
                                         inline asset get_balance( account_name owner, symbol_name sym )const;

                                         Private:
                                         //table
                                        //accounts table is made up of different account objects each holding the balance for a                  different token
                                        struct account {
                                              asset    balance;

                                              uint64_t primary_key()const { return balance.symbol.name(); }
                                       };
                                       //table
                                       //The stat table is made up of currency_stats objects (defined by struct currency_stats) that holds a supply, a max_supply, and an issuer.
                                       struct currency_stats {
                                               asset          supply;
                                               asset          max_supply;
                                               account_name   issuer;

                                               uint64_t primary_key()const { return supply.symbol.name(); }
                                     };

                                    //TABLE  :-
                                        this contract will hold data into two different scopes. The accounts table is scoped to an eosio account, and the stat table is scoped to a token symbol name.
                                    typedef eosio::multi_index<N(accounts), account> accounts;
                                    typedef eosio::multi_index<N(stat), currency_stats> stats;

                                    //`code` is the name of the account which has write permission and the `scope` is the account where the data gets stored.

                                    void sub_balance( account_name owner, asset value );
                                    void add_balance( account_name owner, asset value, account_name ram_payer );

                                    public:
                                          struct transfer_args {
                                                   account_name  from;
                                                   account_name  to;
                                                   asset         quantity;
                                                   string        memo;
                                          };
                                  };

                                  asset token::get_supply( symbol_name sym )const
                                   {
                                               stats statstable( _self, sym );
                                               const auto& st = statstable.get( sym );
                                               return st.supply;
                                  }

                                 asset token::get_balance( account_name owner, symbol_name sym )const
                                 {
                                          accounts accountstable( _self, owner );
                                          const auto& ac = accountstable.get( sym );
                                          return ac.balance;
                                }


Actions :  Implemented in cpp file :-

        
Create   -create a new token
              Parameters :- Issuer,maximum supply
              Issuer is the only one to increase the token.
              Permissioned operations requires  using the -p flag

                                  /**
                                              *  @file
                                              *  @copyright defined in eos/LICENSE.txt
                                  */

                                 #include "eosio.token.hpp"

                                 namespace eosio {

                                           void token::create( account_name issuer,
                                           asset        maximum_supply )
                                           {
                                                 require_auth( _self );
                                                 //extract the symbol for the maximum_supply asset
                                                 auto sym = maximum_supply.symbol;
                                                 eosio_assert( sym.is_valid(), "invalid symbol name" );
                                                 eosio_assert( maximum_supply.is_valid(), "invalid supply");
                                                 eosio_assert( maximum_supply.amount > 0, "max-supply must be positive");
                                                 // stat table called statstable is constructed using the symbols name (token symbol) as its scope
                                                 stats statstable( _self, sym.name() );
                                                // checks whether token already exists or not.
                                                 auto existing = statstable.find( sym.name() );
                                                 eosio_assert( existing == statstable.end(), "token with symbol already exists" );
                                                 //The first parameter _self in the emplace function means that this contracts account ‘eosio.token’ will pay for the staked storage
                                                statstable.emplace( _self, [&]( auto& s ) {
                                                       s.supply.symbol = maximum_supply.symbol;
                                                       s.max_supply    = maximum_supply;
                                                       s.issuer        = issuer;
                                                });
                                               //supply's symbol is used as the key for locating the table row
                                               }


                                              //params :- tokenReceiver,token Quantity,memo

                                              void token::issue( account_name to, asset quantity, string memo )
                                              {
                                                   //Extract the token Symbol
                                                   auto sym = quantity.symbol;
                                                   eosio_assert( sym.is_valid(), "invalid symbol name" );
                                                   eosio_assert( memo.size() <= 256, "memo has more than 256 bytes" );

                                                    //create the stat table using the symbol name as the scope
                                                     auto sym_name = sym.name();
                                                     stats statstable( _self, sym_name );
                                                     auto existing = statstable.find( sym_name );
                                                     eosio_assert( existing != statstable.end(), "token with symbol does not exist, create token before issue" );
                                                     //st is declared and set to the actual object that the existing iterator is pointing to.
                                                     const auto& st = *existing;
  
                                                     //The issuer for the created token is required to sign the transaction
                                                     require_auth( st.issuer );
                                                     eosio_assert( quantity.is_valid(), "invalid quantity" );
                                                     eosio_assert( quantity.amount > 0, "must issue positive quantity" );

                                                     eosio_assert( quantity.symbol == st.supply.symbol, "symbol precision mismatch" );
                                                     eosio_assert( quantity.amount <= st.max_supply.amount - st.supply.amount, "quantity exceeds available supply");
                                                     //The currency_stats st for our existing token is modified and the issued quantity is added to the supply
                                                     statstable.modify( st, 0, [&]( auto& s ) {
                                                              s.supply += quantity;
                                                      });

                                                     //The issuer will also have this supply added to their balance.
                                                     add_balance( st.issuer, quantity, st.issuer );

                                                     if( to != st.issuer ) {
                                                           //transfer function is called via the SEND_INLINE_ACTION() macro which will transfer the funds.

                                                         //     this                    the contract code the action belongs to
                                                                transfer the anme of the action
                                                                {st.issuer, N(active)} the permissions required for the action
                                                                {st.issuer, to, quantity, memo} the arguments for the action itself

                                                                SEND_INLINE_ACTION( *this, transfer, {st.issuer,N(active)}, {st.issuer, to, quantity, memo} );
    }
                                                         }
                                                       //transfer function

                                                       //from and to accounts. The symbol is extracted from the quantity and used to get the currency_stats for the token.
                                                      void token::transfer( account_name from,
                                                                account_name to,
                                                                asset        quantity,
                                                                string       memo )
                                                       {
                                                            eosio_assert( from != to, "cannot transfer to self" );
                                                            require_auth( from );
                                                            eosio_assert( is_account( to ), "to account does not exist");
                                                            auto sym = quantity.symbol.name();
                                                            stats statstable( _self, sym );
                                                            const auto& st = statstable.get( sym );

                                                           // notify both the sender and receiver about action
                                                           require_recipient( from );  
                                                           require_recipient( to );

                                                           eosio_assert( quantity.is_valid(), "invalid quantity" );
                                                           eosio_assert( quantity.amount > 0, "must transfer positive quantity" );
                                                           eosio_assert( quantity.symbol == st.supply.symbol, "symbol precision mismatch" );
                                                           eosio_assert( memo.size() <= 256, "memo has more than 256 bytes" );


                                                           sub_balance( from, quantity );
                                                           add_balance( to, quantity, from );
                                                          }

                                                         void token::sub_balance( account_name owner, asset value ) {
                                                                accounts from_acnts( _self, owner );

                                                                const auto& from = from_acnts.get( value.symbol.name(), "no balance object found" );
                                                                eosio_assert( from.balance.amount >= value.amount, "overdrawn balance" );


                                                                      if( from.balance.amount == value.amount ) {
                                                                                  from_acnts.erase( from );
                                                                      } else {
                                                                                 from_acnts.modify( from, owner, [&]( auto& a ) {
                                                                                          a.balance -= value;
                                                                               });
                                                                       }
                                                                     }

                                                                  void token::add_balance( account_name owner, asset value, account_name ram_payer )
                                                                   {
                                                                        accounts to_acnts( _self, owner );
                                                                        auto to = to_acnts.find( value.symbol.name() );
                                                                        if( to == to_acnts.end() ) {
                                                                             to_acnts.emplace( ram_payer, [&]( auto& a ){
                                                                             a.balance = value;
                                                                       });
                                                                       } else {
                                                                            to_acnts.modify( to, 0, [&]( auto& a ) {
                                                                                 a.balance += value;
                                                                           });
                                                                       }
                                                                        }

                                                                  } /// namespace eosio

                                                                   EOSIO_ABI( eosio::token, (create)(issue)(transfer) )




References: [EOSIO](https://developers.eos.io/eosio-nodeos/docs)

            [EOSIO Token Contract Reference](https://medium.com/coinmonks/understanding-the-eosio-token-contract-87466b9fdca9)

Contributors:
- [Crissi](https://github.com/crissiaccubits)
- [Goutham](https://github.com/goutham-ab)
