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
  
### For the se
<details>
  <summary>View code</summary>

</details>
