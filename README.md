# dockerized-auction-keeper

`dockerized-auction-keeper` contains a preconfigured [auction-keeper](https://github.com/makerdao/auction-keeper) that follows a simple FMV discount pricing model. With docker as the only prerequisite, this instance is well-suited for first-time auction keeper operators.

Note: Docker image will be created based on current master branch when you first run the keeper. If you want to rebuild image
with latest master make sure keepers are stopped then run `./cleanup.sh` script

### Install Prerequisite

- **Install Docker Community Edition for your OS:**
```
https://docs.docker.com/install/
```
- **Install Docker Compose for your OS:**
```
https://docs.docker.com/compose/install/
```

### Setup flip keepers

After following the setup procedure below, this keeper works out of the box under the following configuration:
- Participates in up to 100 active ETH-A / ETH-B Flip auctions; it does not start new ones
- Begins scan at a prescribed auction id - we recommend starting at:
  - `mainnet` - 4500
  - `kovan` - 1800
- Looks for Vaults (i.e. `urns`) at a supplied block height - we recommend starting at the block that `Vat` was deployed:
  - `mainnet` - 8928152
  - `kovan 1.0.2` - 14764534
- Uses a pricing model that tracks the price of ETH / BAT via a public API and applies a `DISCOUNT` before participating
- All logs from the keeper are saved and appended to a single file (`auction-keeper-flip-ETH-A.log` or `auction-keeper-flip-BAT-A.log`)

- Place unlocked keystore and password file for account address under `secrets` directory. The names of the keystore and password files will need to be updated in the `FLIP_ETH_A_ACCOUNT_KEY` in the env.
- Configure following variables in `env/environment.sh` file:
    - `SERVER_ETH_RPC_HOST`: URL to ETH Parity node
    - `SERVER_ETH_RPC_PORT`: ETH RPC port
    - `ETHGASSTATION_API_KEY`: eth gas station API KEY, can be applied for at https://data.concourseopen.com/
    - `GASPRICE_MULTIPLIER`: dynamic gas multiplier (e.g. if 2.0 then will use 2 * base)
    - `FIRST_BLOCK_TO_CHECK`: Recommendation under introduction section
    - `FLIP_ACCOUNT_ADDRESS`: address to use for bidding
    - `FLIP_ETH_A_ACCOUNT_KEY` and `FLIP_BAT_A_ACCOUNT_KEY`: account key format of `key_file=/opt/keeper/secrets/keystore.json,pass_file=/opt/keeper/secrets/password.txt`
    Note: path to file should always be `/opt/keeper/secrets/` followed by the name of file you create under secrets directory
    Ex: if you put `keystore-flip-a.json` and `password-flip-a.txt` under `secrets` directory then var should be configured as
    `FLIP_ETH_A_ACCOUNT_KEY='key_file=/opt/keeper/secrets/keystore-flip-a.json,pass_file=/opt/keeper/secrets/password-flip-a.txt'`
    or
    `FLIP_BAT_A_ACCOUNT_KEY='key_file=/opt/keeper/secrets/keystore-flip-a.json,pass_file=/opt/keeper/secrets/password-flip-a.txt'`
    - `FLIP_DAI_IN_VAT`: Amount of Dai in Vat (Internal Dai Balance); important that this is higher than your largest estimated bid amount
    - `FLIP_MINIMUM_AUCTION_ID_TO_CHECK`: Recommendation under introduction section
    - `FLIP_ETH_DISCOUNT` and `FLIP_BAT_DISCOUNT`: Discount from ETH's or BAT's FMV, which will be used as the bid price

### Setup flop keeper

- Place unlocked keystore and password file for account address under `secrets` directory. The names of the keystore and password files will need to be updated in the `FLOP_ACCOUNT_KEY` in the env.
- Configure following variables in `env/environment.sh` file:
    - `SERVER_ETH_RPC_HOST`: URL to ETH Parity node
    - `SERVER_ETH_RPC_PORT`: ETH RPC port
    - `ETHGASSTATION_API_KEY`: eth gas station API KEY, can be applied for at https://data.concourseopen.com/
    - `GASPRICE_MULTIPLIER`: dynamic gas multiplier (e.g. if 2.0 then will use 2 * base)
    - `FIRST_BLOCK_TO_CHECK`: Recommendation under introduction section
    - `FLOP_ACCOUNT_ADDRESS`: address to use for bidding
    - `FLOP_ACCOUNT_KEY`: account key format of `key_file=/opt/keeper/secrets/keystore.json,pass_file=/opt/keeper/secrets/password.txt`
    Note: path to file should always be `/opt/keeper/secrets/` followed by the name of file you create under secrets directory
    Ex: if you put `keystore-flop.json` and `password-flop.txt` under `secrets` directory then var should be configured as
    `FLOP_ACCOUNT_KEY='key_file=/opt/keeper/secrets/keystore-flop.json,pass_file=/opt/keeper/secrets/password-flop.txt'`
    - `FLOP_DAI_IN_VAT`: Amount of Dai in Vat (Internal Dai Balance); important that this is higher than your largest estimated bid amount
    - `FLOP_MKR_DISCOUNT`: Discount from MKR's FMV, which will be used as the bid price

### Run

flip-eth-a keeper
`./start-keeper.sh flip-eth-a | tee -a -i auction-keeper-flip-ETH-A.log`

flip-bat-a keeper
`./start-keeper.sh flip-bat-a | tee -a -i auction-keeper-flip-BAT-A.log`

flop keeper
`./start-keeper.sh flop | tee -a -i auction-keeper-flop.log`

### Shutdown
This will gracefully stop keeper and will exit DAI / collateral from Vat contract to keeper operating address

flip-eth-a keeper
`./stop-keeper.sh flip-eth-a`

flip-bat-a keeper
`./stop-keeper.sh flip-bat-a`

flop keeper
`./stop-keeper.sh flop`

### Using a dynamic gas price model for bids

- configure following variables in `env/environment.sh` file:
```
# Dynamic Gas Price Model
# 0 = get the gas price from the node (default)
# 1 = get the gas price from ethgasstation.info
#     must set ETHGASSTATION_API_KEY
# 2 = get the gas price from etherchain.org
GAS_MODE=0
# increase this if you want to use higher price than the one reported
# (e.g. if 2.0 then will use 2 * fastest)
GASPRICE_MULTIPLIER=1.4
# ETHGASSTATION_API_KEY is optional.  If you fill it in the model will use
# ethgasstation.info for dynamic gas, otherwise we will simply check the node.
ETHGASSTATION_API_KEY=MY_ETH_GASSTATION_KEY
```

### Optional additions

Other auction keepers can be added in `docker-compose.yml` e.g. for an USDC flipper
- add startup scripts `flip-usdc.sh` and `model-usdc.sh` files under `flip/usdc` directory
- configure any new env vars used by startup scripts in `env/environment.sh` script (e.g. `ACCOUNT_FLIP_USDC_KEY`)
- add keeper in `docker-compose.yml`
```
  flip-usdc:
    build: .
    image: makerdao/auction-keeper
    container_name: flip-usdc
    working_dir: /opt/keeper/flip-usdc/
    volumes:
      - $PWD/secrets:/opt/keeper/secrets
      - $PWD/env/:/opt/keeper/env/
      - $PWD/flip/usdc/:/opt/keeper/usdc/
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "10"
    command: ./flip-usdc.sh model-usdc.sh
```
- start it as `./start-keeper.sh flip-usdc | tee -a -i auction-keeper-flip-USDC.log`

## License
See [COPYING](https://github.com/makerdao/dockerized-auction-keeper/blob/master/COPYING) file.

## Disclaimer
YOU (MEANING ANY INDIVIDUAL OR ENTITY ACCESSING, USING OR BOTH THE SOFTWARE INCLUDED IN THIS GITHUB REPOSITORY) EXPRESSLY UNDERSTAND AND AGREE THAT YOUR USE OF THE SOFTWARE IS AT YOUR SOLE RISK. THE SOFTWARE IN THIS GITHUB REPOSITORY IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE. YOU RELEASE AUTHORS OR COPYRIGHT HOLDERS FROM ALL LIABILITY FOR YOU HAVING ACQUIRED OR NOT ACQUIRED CONTENT IN THIS GITHUB REPOSITORY. THE AUTHORS OR COPYRIGHT HOLDERS MAKE NO REPRESENTATIONS CONCERNING ANY CONTENT CONTAINED IN OR ACCESSED THROUGH THE SERVICE, AND THE AUTHORS OR COPYRIGHT HOLDERS WILL NOT BE RESPONSIBLE OR LIABLE FOR THE ACCURACY, COPYRIGHT COMPLIANCE, LEGALITY OR DECENCY OF MATERIAL CONTAINED IN OR ACCESSED THROUGH THIS GITHUB REPOSITORY.
