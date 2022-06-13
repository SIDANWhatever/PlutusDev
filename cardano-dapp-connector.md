# Investigating the Cardano Wallet Connector Template
* Repo: https://github.com/dynamicstrategies/cardano-wallet-connector
* Documentation provided by: @SIDANWhatever
* Date: 13 June 2022

## Background
Apart from Plutus scripting, serialization library and wallet connection are crucial for Dapp production. Thanks to @dynamicstrategies, a well built template for Cardano wallet connection is available to Cardano community. This documentation would analyze how each `react.js` items interacting with each other. Understanding the fundamental logics help build dynamic Dapp on Cardano.

## Script Analysis

### Set the different pararmeters as the basic props of the connector components, which would then be updated upon transactions. (`constructor(props)`)
1. `this.state` stores the transaction details
2. `this.API` stores the API of wallet chosen
3. `this.protocolParams` stores the protocol parameters needed for every transaction to be initiated.

<details>
  <summary>View code</summary>
  
```react.js
constructor(props)
{
    super(props);

    this.state = {
        selectedTabId: "1",
        whichWalletSelected: undefined,
        walletFound: false,
        walletIsEnabled: false,
        walletName: undefined,
        walletIcon: undefined,
        walletAPIVersion: undefined,
        wallets: [],

        networkId: undefined,
        Utxos: undefined,
        CollatUtxos: undefined,
        balance: undefined,
        changeAddress: undefined,
        rewardAddress: undefined,
        usedAddress: undefined,

        txBody: undefined,
        txBodyCborHex_unsigned: "",
        txBodyCborHex_signed: "",
        submittedTxHash: "",

        addressBech32SendADA: "addr_test1qrt7j04dtk4hfjq036r2nfewt59q8zpa69ax88utyr6es2ar72l7vd6evxct69wcje5cs25ze4qeshejy828h30zkydsu4yrmm",
        lovelaceToSend: 3000000,
        assetNameHex: "4c494645",
        assetPolicyIdHex: "ae02017105527c6c0c9840397a39cc5ca39fabe5b9998ba70fda5f2f",
        assetAmountToSend: 5,
        addressScriptBech32: "addr_test1wpnlxv2xv9a9ucvnvzqakwepzl9ltx7jzgm53av2e9ncv4sysemm8",
        datumStr: "12345678",
        plutusScriptCborHex: "4e4d01000033222220051200120011",
        transactionIdLocked: "",
        transactionIndxLocked: 0,
        lovelaceLocked: 3000000,
        manualFee: 900000,

    }

    this.API = undefined;

    this.protocolParams = {
        linearFee: {
            minFeeA: "44",
            minFeeB: "155381",
        },
        minUtxo: "34482",
        poolDeposit: "500000000",
        keyDeposit: "2000000",
        maxValSize: 5000,
        maxTxSize: 16384,
        priceMem: 0.0577,
        priceStep: 0.0000721,
        coinsPerUtxoWord: "34482",
    }

    this.pollWallets = this.pollWallets.bind(this);
}
```
</details>

### Pull the wallet to be connected (`pollWallets`)

This could check if any wallets are connected with the browser and add them to the list. Then it would default choose the first wallet identified to request connection.

<details>
  <summary>View code</summary>

```react.js
pollWallets = (count = 0) => {
    const wallets = [];
    for(const key in window.cardano) {
        if (window.cardano[key].enable && wallets.indexOf(key) === -1) {
            wallets.push(key);
        }
    }
    if (wallets.length === 0 && count < 3) {
        setTimeout(() => {
            this.pollWallets(count + 1);
        }, 1000);
        return;
    }
    this.setState({
        wallets,
        whichWalletSelected: wallets[0]
    }, () => {
        this.refreshData()
    });
}
```

</details>
  
### Update selected wallet state value (`handleWalletSelect`)
  

<details>
  <summary>View code</summary>

  ```react.js
  handleWalletSelect = (obj) => {
    const whichWalletSelected = obj.target.value;
    this.setState({ whichWalletSelected }, () => {
      this.refreshData();
    });
  };
  ```
</details>
  
### Generate address from the plutus contract cborhex (`generateScriptAddress`)

This function is used for demonstrating how the script address generated through `cardano-cli` and the `cardano-serialisation-lib` are indeed the same. With the initial input of the `cborHex` in the plutus script (`.plutus` file).
  
<details>
  <summary>View code</summary>
  
  ```react.js
  generateScriptAddress = () => {

    const script = PlutusScript.from_bytes(
      Buffer.from(this.state.plutusScriptCborHex, "hex")
    );
    const blake2bhash = blake.blake2b(script.to_bytes(), 0, 28);
    const scripthash = ScriptHash.from_bytes(Buffer.from(blake2bhash, "hex"));

    const cred = StakeCredential.from_scripthash(scripthash);
    const networkId = NetworkInfo.testnet().network_id();
    const baseAddr = EnterpriseAddress.new(networkId, cred);
    const addr = baseAddr.to_address();
    const addrBech32 = addr.to_bech32();
  };
  ```
</details>

### For the selected wallet, check if it is running (`checkIfWalletFound`)
<details>
  <summary>View code</summary>

  ```react.js
  checkIfWalletFound = () => {
    const walletKey = this.state.whichWalletSelected;
    const walletFound = !!window?.cardano?.[walletKey];
    this.setState({ walletFound });
    return walletFound;
  };
  ```  

</details>
  

### Check if the selected wallet is enabled (`checkIfWalletEnabled`)
<details>
  <summary>View code</summary>

```react.js
checkIfWalletEnabled = async () => {
  let walletIsEnabled = false;

  try {
    const walletName = this.state.whichWalletSelected;
    walletIsEnabled = await window.cardano[walletName].isEnabled();
  } catch (err) {
    console.log(err);
  }
  this.setState({ walletIsEnabled });

  return walletIsEnabled;
};
```
  
</details>
  
### The actual function to enable the wallet (`enableWallet`)
<details>
  <summary>View code</summary>

```react.js
enableWallet = async () => {
  const walletKey = this.state.whichWalletSelected;
  try {
    this.API = await window.cardano[walletKey].enable();
  } catch (err) {
    console.log(err);
  }
  return this.checkIfWalletEnabled();
};
```
</details>
  
### Get the wallet API from the wallet provider and write into state (`getAPIVersion`)
<details>
  <summary>View code</summary>

  ```react.js
  getAPIVersion = () => {
    const walletKey = this.state.whichWalletSelected;
    const walletAPIVersion = window?.cardano?.[walletKey].apiVersion;
    this.setState({ walletAPIVersion });
    return walletAPIVersion;
  };
  ```
</details>

### Get the wallet name from the wallet provider and write into state (`getWalletName`)
<details>
  <summary>View code</summary>

  ```react.js
  getWalletName = () => {
    const walletKey = this.state.whichWalletSelected;
    const walletName = window?.cardano?.[walletKey].name;
    this.setState({ walletName });
    return walletName;
  };
  ```
  
</details>

### Gets the Network ID to which the wallet is connected (`getNetworkId`)
  
<details>
  <summary>View code</summary>

  ```react.js
  getNetworkId = async () => {
    try {
      const networkId = await this.API.getNetworkId();
      this.setState({ networkId });
    } catch (err) {
      console.log(err);
    }
  };
  ```
  
</details>

### Get the utxo from user's wallet and ready for `tx-in` entry for transaction submission (`getUtxos`)
  
<details>
  <summary>View code</summary>

  ```react.js
  getUtxos = async () => {
    let Utxos = [];

    try {
      const rawUtxos = await this.API.getUtxos();

      for (const rawUtxo of rawUtxos) {
        const utxo = TransactionUnspentOutput.from_bytes(
          Buffer.from(rawUtxo, "hex")
        );
        const input = utxo.input();
        const txid = Buffer.from(
          input.transaction_id().to_bytes(),
          "utf8"
        ).toString("hex");
        const txindx = input.index();
        const output = utxo.output();
        const amount = output.amount().coin().to_str(); // ADA amount in lovelace
        const multiasset = output.amount().multiasset();
        let multiAssetStr = "";

        if (multiasset) {
          const keys = multiasset.keys(); // policy Ids of thee multiasset
          const N = keys.len();
          // console.log(`${N} Multiassets in the UTXO`)

          for (let i = 0; i < N; i++) {
            const policyId = keys.get(i);
            const policyIdHex = Buffer.from(
              policyId.to_bytes(),
              "utf8"
            ).toString("hex");
            // console.log(`policyId: ${policyIdHex}`)
            const assets = multiasset.get(policyId);
            const assetNames = assets.keys();
            const K = assetNames.len();
            // console.log(`${K} Assets in the Multiasset`)

            for (let j = 0; j < K; j++) {
              const assetName = assetNames.get(j);
              const assetNameString = Buffer.from(
                assetName.name(),
                "utf8"
              ).toString();
              const assetNameHex = Buffer.from(
                assetName.name(),
                "utf8"
              ).toString("hex");
              const multiassetAmt = multiasset.get_asset(policyId, assetName);
              multiAssetStr += `+ ${multiassetAmt.to_str()} + ${policyIdHex}.${assetNameHex} (${assetNameString})`;
              // console.log(assetNameString)
              // console.log(`Asset Name: ${assetNameHex}`)
            }
          }
        }

        const obj = {
          txid: txid,
          txindx: txindx,
          amount: amount,
          str: `${txid} #${txindx} = ${amount}`,
          multiAssetStr: multiAssetStr,
          TransactionUnspentOutput: utxo,
        };
        Utxos.push(obj);
        // console.log(`utxo: ${str}`)
      }
      this.setState({ Utxos });
    } catch (err) {
      console.log(err);
    }
  };
  ```
  
</details>
  
### Get the collateral needed for transaction submission (`getCollateral`)
  
<details>
  <summary>View code</summary>

  ```react.js
  getCollateral = async () => {
    let CollatUtxos = [];

    try {
      let collateral = [];

      const wallet = this.state.whichWalletSelected;
      if (wallet === "nami") {
        collateral = await this.API.experimental.getCollateral();
      } else {
        collateral = await this.API.getCollateral();
      }

      for (const x of collateral) {
        const utxo = TransactionUnspentOutput.from_bytes(Buffer.from(x, "hex"));
        CollatUtxos.push(utxo);
        // console.log(utxo)
      }
      this.setState({ CollatUtxos });
    } catch (err) {
      console.log(err);
    }
  };
  ```
  
</details>

### Get the ADA balance from user's wallet (`getBalance`)
  
<details>
  <summary>View code</summary>

  ```react.js
  getBalance = async () => {
    try {
      const balanceCBORHex = await this.API.getBalance();

      const balance = Value.from_bytes(Buffer.from(balanceCBORHex, "hex"))
        .coin()
        .to_str();
      this.setState({ balance });
    } catch (err) {
      console.log(err);
    }
  };
  ```
  
</details>

### Get the address where the spare UTxO is sent after transaction (`getChangeAddress`)
  
<details>
  <summary>View code</summary>

  ```react.js
  getChangeAddress = async () => {
    try {
      const raw = await this.API.getChangeAddress();
      const changeAddress = Address.from_bytes(
        Buffer.from(raw, "hex")
      ).to_bech32();
      this.setState({ changeAddress });
    } catch (err) {
      console.log(err);
    }
  };
  ```
  
</details>
  
### Get the reward address where the staking rewards shd be claim to (`getRewardAddresses`)
  
<details>
  <summary>View code</summary>

  ```react.js
  getRewardAddresses = async () => {
    try {
      const raw = await this.API.getRewardAddresses();
      const rawFirst = raw[0];
      const rewardAddress = Address.from_bytes(
        Buffer.from(rawFirst, "hex")
      ).to_bech32();
      // console.log(rewardAddress)
      this.setState({ rewardAddress });
    } catch (err) {
      console.log(err);
    }
  };
  ```
  
</details>
  
### Get the used wallet address for unknown reason (`getUsedAddresses`)
  
<details>
  <summary>View code</summary>

  ```react.js
  getUsedAddresses = async () => {
    try {
      const raw = await this.API.getUsedAddresses();
      const rawFirst = raw[0];
      const usedAddress = Address.from_bytes(
        Buffer.from(rawFirst, "hex")
      ).to_bech32();
      // console.log(rewardAddress)
      this.setState({ usedAddress });
    } catch (err) {
      console.log(err);
    }
  };
  ```
  
</details>
  
### Refreshing data for collecting information from user's wallet API (`refreshData`)
  
<details>
  <summary>View code</summary>

  ```react.js
  refreshData = async () => {
    this.generateScriptAddress();

    try {
      const walletFound = this.checkIfWalletFound();
      if (walletFound) {
        await this.getAPIVersion();
        await this.getWalletName();
        const walletEnabled = await this.enableWallet();
        if (walletEnabled) {
          await this.getNetworkId();
          await this.getUtxos();
          await this.getCollateral();
          await this.getBalance();
          await this.getChangeAddress();
          await this.getRewardAddresses();
          await this.getUsedAddresses();
        } else {
          await this.setState({
            Utxos: null,
            CollatUtxos: null,
            balance: null,
            changeAddress: null,
            rewardAddress: null,
            usedAddress: null,

            txBody: null,
            txBodyCborHex_unsigned: "",
            txBodyCborHex_signed: "",
            submittedTxHash: "",
          });
        }
      } else {
        await this.setState({
          walletIsEnabled: false,

          Utxos: null,
          CollatUtxos: null,
          balance: null,
          changeAddress: null,
          rewardAddress: null,
          usedAddress: null,

          txBody: null,
          txBodyCborHex_unsigned: "",
          txBodyCborHex_signed: "",
          submittedTxHash: "",
        });
      }
    } catch (err) {
      console.log(err);
    }
  };
  ```
  
</details>

###
  
<details>
  <summary>View code</summary>

  ```react.js
  
  ```
  
</details>
  
###
  
<details>
  <summary>View code</summary>

  ```react.js
  
  ```
  
</details>
