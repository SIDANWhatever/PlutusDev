# Plutus Development Using Plutus-starter Template
* Plutus-starter: https://github.com/input-output-hk/plutus-starter
* Guide: https://github.com/plutus-community/docs/blob/main/docs/Guides/plutus-starter.md

# Alternative: Build from Cabal Module
* Please follow below's set up for dev environemnt.

## 0. Have all the dependencies / nix-shell set up.
* Refer to https://github.com/SIDANWhatever/PlutusPioneerProgramme/blob/main/MacM1-Setup.md & https://github.com/SIDANWhatever/PlutusPioneerProgramme/blob/main/Week01-code-summary.md for details.

## 1. Clone the plutus apps repo from IOHK
>```
>git clone https://github.com/input-output-hk/plutus-apps.git
>git checkout c9c1e917edbfa3b972c92108d7b94d5430e07a28
>```
* We use PPP cohort 3 lecture 8's tag to make sure the plutus-apps nix-shell works.

## 2. Enter the nix-shell
>```
>cd plutus-apps
>nix-shell
>```
* Please expect build time here.

## 3. Cabal config
* Create the `.hs` file (script file) inside `/app` directory
* Replace the `library` inside the `.cabal` file with the below:
>```cabal
>common common-all
>    default-language: Haskell2010
>    ghc-options:      -Wall
>
>library
>    import:            common-all                   
>    exposed-modules:   example
>                       testing
>                       -- Put your .hs file name here, e.g. for example.hs & testing.hs inside /app you should put as above
>    build-depends:     aeson
>                     , base ^>=4.14.1.0
>                     , deepseq >= 1.4.4.0
>                     , bytestring
>                     , cardano-api
>                     , cardano-crypto-class
>                     , cardano-ledger-core
>                     , cardano-ledger-shelley
>                     , cardano-wallet-core
>                     , containers
>                     , data-default
>                     , freer-extras
>                     , openapi3
>                     , playground-common
>                     , plutus-contract
>                     , plutus-ledger
>                     , plutus-ledger-api
>                     , plutus-ledger-constraints
>                     , plutus-pab
>                     , plutus-tx-plugin
>                     , plutus-tx
>                     , plutus-use-cases
>                     , prettyprinter
>                     , serialise
>                     , text
>    hs-source-dirs:   src
>    ghc-options:     -Wall -fobject-code -fno-ignore-interface-pragmas -fno-omit-interface-pragmas -fno-strictness -fno-spec-constr -fno-specialise
>

* For each `.hs` code, add one paragrah as below in the `.cabal` file
>```cabal
>executable Example
>    import:           common-all
>    main-is:          example.hs
>    other-modules:    NFT
>    build-depends:    base ^>=4.14.1.0,
>                      deepseq >= 1.4.4.0
>    hs-source-dirs:   app, src
>```
