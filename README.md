## Mining stratum for Komodo and Komodo assetchains.

Requirements
------------
* node v10+
* libsodium
* boost
* Redis (see https://redis.io/topics/quickstart for details)

Differences between this and Z-NOMP
------------
* This is meant for Komodo mining
* Founders, Treasury, and other ZEC/ZEN specific stuff is removed

Upgrade
-------------
Please be sure to backup your `./coins` and `./pool_configs` directory before upgrading

Kill your running pool (CTRL-C)
```shell
cd knomp
git pull
npm install
npm start
```

Install Daemon
-------------
Some initial setup
```shell
# The following packages are needed to build both Komodo and this stratum:
sudo apt-get update
sudo apt-get install build-essential pkg-config libc6-dev m4 g++-multilib autoconf libtool ncurses-dev unzip git python python-zmq zlib1g-dev wget libcurl4-openssl-dev bsdmainutils automake curl libboost-dev libboost-system-dev libsodium-dev jq redis-server nano -y
```
Now, let's build Komodo
```shell
git clone https://github.com/jl777/komodo -b FSM
cd komodo
zcutil/fetch-params.sh
zcutil/build.sh -j8
```

We need to generate the coins files (coin daemon must be running!): `gencfg.sh <coin name>`

You can run just gencfg.sh with no coin name to use the assetchains.json in komodo/src directory for all coins. Make sure you edit the template with the correct values you want before running the config generator.

We need node and npminstalled
=======
strip src/komodod
strip src/komodo-cli
```

Now, let's run the assets. This will start ALL of the assets might take a day or so to sync, depending on system speed/network connection.

_If you are setting up a single chain to mine and/or don't know what pubkey is, skip this step and use the startup params for the komodod daemon as provided by the individual coin's team._
```shell
cd ~/komodo/src
./assetchains
```

Install Pool
-------------
Once all the chains you want on your pool have synced up we can configure the stratum.

We need node and npm installed

```shell
cd ~
curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
```

Now, let's build our stratum and run it. This will install the pool and configure it for all the assetchains on your system automatically. It must be run from the same user as the coin deamons were launched, as it pulls the rpcuser/pass from the conf file in the home directory.
```shell
git clone https://github.com/blackjok3rtt/knomp
cd knomp
nano gencfg.sh
```
Edit line 3 in so that it has your own KMD based address, CTRL-X then Y to save and exit
```shell
./gencfg.sh
npm install
cp config_example.json config.json (and configure it)
npm start
```


## Disable Coinbase Mode 
This mode is enabled by default in the coins.template with`"disablecb" : true` 

To disable it, change the value to false. This mode uses -pubkey to tell the daemon where the coinbase should be sent, and uses the daemons coinbase transaction rather then having the pool create the coinabse transaction. This enables special coinbase transactions, such as ac_founders and ac_script or new modes with CC vouts in the coinbase not yet created, it will work with all coins, except Full Z support described below. 

The pool fee is taken in the payment processor using this mode, and might not be 100% accurate down to the single satoshi, so the pool address may end up with some small amount of coins over time. To use the pool fee, just change `rewardRecipents` value in the `poolconfig.template` before running the `gencfg.sh` script as you normally would for the standard mode.


Full Z Transaction Support

If all went well the program should start without error and you should be able to browse to your pool website on your server via port 8080.

Payment Processing
-------------
Please note that the default configs generated are for solo mining. If you wish to create a public pool please modify the configs like in this [example config](https://github.com/z-classic/z-nomp/blob/master/pool_configs/komodo_example.json)

There is now a config option you can add to your pool_configs/coin.json to toggle making an attempt at a payment upon pool startup.

```
"paymentProcessing": {
    "payOnStart": true,
    ...
}
```

Invalid Worker Addresses
-------------
You can add an option to your pool_config to have any miners that mine with an invalid address (if they somehow get through) to pay out to an address of your choosing
```
"invalidAddress":"zsValidAddressOfYourChoosingThatsNotThePoolZAddress"
```

Full Z Transaction Support (Sprout)

-------------
This is an option to force miners to use a Z address as their username for payouts

In your coins file add: 
```
"privateChain": true,
"burnFees": true
```

Sapling and Sapling Payment Support
-------------
In coins/pirate.json file:
```
"privateChain": true,
"burnFees": true,
"sapling": 152855
```
Please note, PIRATE sapling became active around 2018-12-15 01:15UTC at block 152855 Now that this has passed this can just be set to `"sapling":true` 

In pool_config:
```
"zAddress": "zsPoolsSaplingAddress",
"walletInterval": 2,
"validateWorkerUsername": true,
"paymentProcessing": {
        "minConf": 5,
        "paymentInterval": 180,
        "maxBlocksPerPayment": 20,
```

More Resources
-------------
[Further info on config](https://github.com/zone117x/node-open-mining-portal#2-configuration) and some [sample configs](https://github.com/z-classic/z-nomp)

License
-------------
Forked from ComputerGenie repo (deleted)

Released under the GNU General Public License v2
http://www.gnu.org/licenses/gpl-2.0.html

_Forked from [z-classic/z-nomp](https://github.com/z-classic/z-nomp) which is incorrectly licensed under MIT License - see [zone117x/node-open-mining-portal](https://github.com/zone117x/node-open-mining-portal)_ 
