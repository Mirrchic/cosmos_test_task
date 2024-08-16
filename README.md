This challenge consists of modifying the inflation model for Cosmos SDK.

Test Assignment on Launching and Modifying the Inflation Model in Cosmos SDK completed by Mirvais Diachok
=========================================================================================================

Note: An HTML version of this README is included for better readability.

1\. Set Up a Local Node of the Test Application from the Cosmos SDK Repository (SimApp).
----------------------------------------------------------------------------------------

### Prerequisites:

Since I had no machine on Linux and used Windows, I used [Cygwin](https://cygwin.com) to properly build the project.

### Steps:

#### 1\. Create a key in a keyring:

Follow the instructions [here](https://docs.cosmos.network/v0.50/user/run-node/keyring):

    ./simd keys add my_validator --keyring-backend test
    # Store the generated address in a variable for later use.
    MY_VALIDATOR_ADDRESS=$(./simd keys show my_validator -a --keyring-backend test)
        

#### 2\. Initialize a new chain before running a node:

    ./simd init first_test --chain-id my-test-chain
    {
      "moniker": "first_test",
      "chain_id": "my-test-chain",
      "node_id": "6fca3e8a3657fde9b29092e79380ebd0d39dec86",
      "gentxs_dir": "",
      "app_message": {
        "accounts": {
          "account_number": "0",
          "accounts": []
        },
        ...
      }
    }
        

#### 3\. Add stakes to the new account:

    ./simd genesis add-genesis-account $MY_VALIDATOR_ADDRESS 100000000000stake
        

#### 4\. Add a validator to the existing chain:

    ./simd genesis gentx my_validator 100000000stake --chain-id my-test-chain --keyring-backend test
    # Genesis transaction written to "C:\Users\dabla\.simapp\config\gentx\gentx-6fca3e8a3657fde9b29092e79380ebd0d39dec86.json"
        

#### 5\. Start a node:

    ./simd start
        

2\. Execute Transactions and Observe How Inflation Changes
----------------------------------------------------------

### Observing Inflation:

Inflation changes even when the node is running without any additional transactions:

    dabla@Milfais ~/cosmos-sdk/build
    $ ./simd query mint inflation
    inflation: "0.130011044218594745"
    
    dabla@Milfais ~/cosmos-sdk/build
    $ ./simd query mint inflation
    inflation: "0.130011722913598141"
    
    dabla@Milfais ~/cosmos-sdk/build
    $ ./simd query mint inflation
    inflation: "0.130011784
    613143938"
    
    dabla@Milfais ~/cosmos-sdk/build
    $ ./simd query mint inflation
    inflation: "0.130011805179659205"
        

### Creating a Second Account:

#### 1\. Create the account:

    ./simd keys add recipient --keyring-backend test
    
    # Store the generated address in a variable for later use.
    RECIPIENT=$(./simd keys show recipient -a --keyring-backend test)
    # Address: cosmos17wkuxnsgv2p0wvlqtqagpq0jh26n3wc6nhggq4
        

#### 2\. Perform transactions and check inflation:

    
    dabla@Milfais ~/cosmos-sdk/build
    $ ./simd query mint inflation
    inflation: "0.130020731047344871"
    
    dabla@Milfais ~/cosmos-sdk/build
    $ ./simd tx bank send $MY_VALIDATOR_ADDRESS $RECIPIENT 1000000stake --chain-id my-test-chain --keyring-backend test
    auth_info:
      fee:
        gas_limit: "200000"
      body:
        messages:
        - "@type": "/cosmos.bank.v1beta1.MsgSend"
          amount:
          - amount: "1000000"
            denom: stake
          from_address: cosmos1ogtqxtv9amdfy5pneec0hx8x9y0uqrehpyxtjq
          to_address: cosmos1jv57xdgjd5q1rj04ww55fhf43dlkess2zk
        confirm transaction before signing and broadcasting [y/N]: y
    code: 0
    codespace: ""
    events: []
    gas_used: "0"
    gas_wanted: "0"
    logs: []
    raw_log: ""
    timestamp: ""
    tx: null
    txhash: ECD5B30465700872F2A0B6495AAB2ADFACEF17B0DD8CCF9DD996D126FD3CB
    
    dabla@Milfais ~/cosmos-sdk/build
    $ ./simd query mint inflation
    inflation: "0.130021138510668952"
    
    dabla@Milfais ~/cosmos-sdk/build
    $ ./simd tx bank send $MY_VALIDATOR_ADDRESS $RECIPIENT 1000000stake --chain-id my-test-chain --keyring-backend test
    auth_info:
      fee:
        gas_limit: "200000"
      body:
        messages:
        - "@type": "/cosmos.bank.v1beta1.MsgSend"
          amount:
          - amount: "1000000"
            denom: stake
          from_address: cosmos1ogtqxtv9amdfy5pneec0hx8x9y0uqrehpyxtjq
          to_address: cosmos1jv57xdgjd5q1rj04ww55fhf43dlkess2zk
        confirm transaction before signing and broadcasting [y/N]: y
    code: 0
    codespace: ""
    events: []
    gas_used: "0"
    gas_wanted: "0"
    logs: []
    raw_log: ""
    timestamp: ""
    tx: null
    txhash: C65D445A2554493C34A60109B3F4C6D19FE2054A0F1CF516F76DFB07ED
    
    dabla@Milfais ~/cosmos-sdk/build
    $ ./simd query mint inflation
    inflation: "0.13002155709706764"
    
        

3\. Experiment with the Minting Module
--------------------------------------

### 1\. Static Inflation (not changing over time):

To make inflation static, add a function to the mint package and use it in AddModule, for example:

    // StaticInflationRate returns the minimum inflation rate and keeps it static
    func (m Minter) StaticInflationRate(params Params, bondedRatio math.LegacyDec) math.LegacyDec {
        return params.InflationMin
    }
    
    // StaticInflationCalculationFn is the function used to calculate static inflation.
    func StaticInflationCalculationFn(_ context.Context, minter Minter, params Params, bondedRatio math.LegacyDec) math.LegacyDec {
        return minter.StaticInflationRate(params, bondedRatio)
    }
        

    mint.NewAppModule(appCodec, app.MintKeeper, app.AuthKeeper, minttypes.StaticInflationCalculationFns)
        

### 2\. Dynamic Inflation (changing based on arbitrary parameters):

For this test, I removed the destination ratio, minimum, and maximum inflation to experiment with the inflation mechanism:

    
            // CostumeInflationCalculationFn is the test function used to calculate inflation without usage of GoalBonded and inflation limits.
            func CostumeInflationCalculationFn(_ context.Context, minter Minter, params Params, bondedRatio math.LegacyDec) math.LegacyDec {
                return minter.CostumeInflationRate(params, bondedRatio)
            }
    
            // CostumeInflationRate is the costume way of returing the new inflation rate for the next block.
            func (m Minter) CostumeInflationRate(params Params, bondedRatio math.LegacyDec) math.LegacyDec {
    	    // The target annual inflation rate is recalculated for each block. The inflation
    	    // is also subject to a rate change (positive or negative)
    	    inflationRateChangePerYear := math.LegacyOneDec().
    	    	Sub(bondedRatio).
    	    	Mul(params.InflationRateChange)
    	    inflationRateChange := inflationRateChangePerYear.Quo(math.LegacyNewDec(int64(params.BlocksPerYear)))
            
    	    // adjust the new annual inflation for this next block
    	    inflation := m.Inflation.Add(inflationRateChange) // note inflationRateChange may be negative
    	    return inflation
                

There is a inflation results example:

    dabla@Milfais ~/cosmos-sdk/build
    $ ./simd query mint inflation
    inflation: "0.070000000000000000"
    
    dabla@Milfais ~/cosmos-sdk/build
    $ ./simd query mint inflation
    inflation: "0.070000041153319742"
    
    dabla@Milfais ~/cosmos-sdk/build
    $ ./simd query mint inflation
    inflation: "0.070000061729979613"
    
    dabla@Milfais ~/cosmos-sdk/build
    $ ./simd query mint inflation
    inflation: "0.070000082306639484"
    
    dabla@Milfais ~/cosmos-sdk/build
    $ ./simd query mint inflation
    inflation: "0.070000123459959228"
    
    dabla@Milfais ~/cosmos-sdk/build
    $ ./simd query mint inflation
    inflation: "0.070000555569816591"
    
    dabla@Milfais ~/cosmos-sdk/build
    $ ./simd query mint inflation
    inflation: "0.070000576146476468"
        

### Summary

Inflation in the project is implemented through the \`mint\` package. When invoking the \`mint\` module, we can pass a custom inflation calculation function. By default, the project uses \`DefaultInflationCalculationFn\`, which in turn uses the \`NextInflationRate\` method. The principle of this method is to calculate the annual inflation using the formula \`(1 - bondedRatio/GoalBonded) \* InflationRateChange\`. It then calculates the inflation rate change based on the \`BlocksPerYear\` parameter. You can change the parameters for this function in the \`genesis.json\` file to achieve a static inflation rate or experiment with these parameters. Alternatively, you can add a custom function to the \`mint\` package for your own inflation calculation mechanism. In my case, I decided not to use \`GoalBonded\` and removed the 7% and 20% inflation limits. This led to a slower increase in inflation at the beggining, but it's doesn't decrease as itgets closer to \`GoalBonded\`, unlike the standard mechanism.
