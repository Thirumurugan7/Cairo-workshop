# Cairo-workshop

to install starkli:

<code>curl https://get.starkli.sh | sh</code>

then close and restart the terminal

then run : <code> starkliup</code>

on successfull installation you can check the version by running the following command:

<code>starkli --version </code>

Create bravoos wallet and take public and private create
create a signer by the following command:

<code>starkli signer keystore from-key \
~/.starkli-wallets/deployer/keystore.json
</code>

It will ask for private key  and password and will populate the keystore.json
Also keep smartcontract class hash and smart wallet address from starkscan

Now we create Account

First we need to create a json file
<code>touch ~/.starkli-wallets/deployer/account.json</code>

then open this file in vim using the following command

<code>vi ~/.starkli-wallets/deployer/account.json
</code>
and paste <code>{
  "version": 1,
  "variant": {
  "type": "open_zeppelin",
  "version": 1,
    "public_key": "<SMART_WALLET_PUBLIC_KEY>"
  },
  "deployment": {
    "status": "deployed",
    "class_hash": "<SMART_WALLET_CLASS_HASH>",
    "address": "<SMART_WALLET_ADDRESS>"
  }
}</code>

save the file using the following command:

<esc>:wq!

Create an Account on infura and Get an API key.
https://app.infura.io/dashboard/ethereum/683c47764f8b4b309533c1b851de0cfc/settings/endpoints
Now let's set the env variables.

First create the env variable file using the following command:

<code>$ touch ~/.starkli-wallets/deployer/envars.sh
</code>
Open the env variable file in vim and paste the following:

export STARKNET_RPC=<API_url_for_starknet_goerli>
export STARKNET_ACCOUNT=~/.starkli-wallets/deployer/account.json
export STARKNET_KEYSTORE=~/.starkli-wallets/deployer/keystore.json

save it by <esc>:wq!

Run the command
<code>$ source ~/.starkli-wallets/deployer/envars.sh</code>

we can check if the env variables are set by echoing it :
echo $STARKNET_RPC $STARKNET_ACCOUNT $STARKNET_KEYSTORE

All set lets install scarb-->

<code>curl --proto '=https' --tlsv1.2 -sSf https://docs.swmansion.com/scarb/install.sh | sh -s -- -v 0.5.1
</code>
on successful install check the version by 
scarb --version

You should get the following:
scarb 0.5.1 (798acce7f 2023-07-05)
cairo: 2.0.1 (https://crates.io/crates/cairo-lang-compiler/2.0.1)

Set-up over

Lets code.
<code>mkdir starknet</code>

<code>scarb new test</code>

code . //to open vs code

in src dir -> create a new file and name it Storage.cairo

in that file enter the following code: 
<code>
#[starknet::interface]
trait simpleStorage<T> {
    fn store(ref self: T, value: u32);
    fn retrieve(self: @T) -> u32;
}

#[starknet::contract]
mod Storage {

    #[storage]
    struct Storage {
        val: u32,
    }

    #[external(v0)]
    impl simpleStorageImpl of super::simpleStorage<ContractState> {
        fn store(ref self: ContractState, value: u32){
            self.val.write(value);
        }

        fn retrieve(self: @ContractState) -> u32 {
            self.val.read()
        }
    }

}

</code>

in lib.cairo

delete everything and add the below code:
<code>
mod Storage;
</code>

in Scarb.toml file add the following code under dependencies:
<code>

starknet = "2.0.1"

[[target.starknet-contract]]
# Enable Sierra codegen.
sierra = true

</code>


and run:
<code>
scarb build
</code>
after successfull Compilation run the following command
<code>
starkli declare ./target/dev/test_Storage.sierra.json
</code>
You will get the class hash
<code>
starkli deploy \<copy_paste_your_class_hash_here>
</code>
You will get contract address



