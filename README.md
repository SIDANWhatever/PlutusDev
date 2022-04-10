# Plutus Development Using Plutus-starter Template
* Plutus-starter: https://github.com/input-output-hk/plutus-starter
* Guide: https://github.com/plutus-community/docs/blob/main/docs/Guides/plutus-starter.md

Note: The guide above demonstrated how you work with built code, heading to production testing. For the start of development, please follow below's step.

## 1. Create a template
* Go to https://github.com/input-output-hk/plutus-starter & click the green button `Use this template`

## 2. Git clone the template to local directory
>```
>git clone https://github.com/SIDANWhatever/plutus-starter
>```

## 3. To add `cardano-cli` and `cardano-node` to the nix-shell
* Change the whole `shell.nix` file's content to below (copy & paste below to the `shell.nix` file)
>```
>{ pure ? false
>, source-repo-override ? { } }:
>let
>  packages = import ./. { inherit source-repo-override; };
>  inherit (packages) pkgs plutus-apps plutus-starter;
>  inherit (plutus-starter) haskell;
>  
>  cardano-node = import
>   (pkgs.fetchgit {
>     url = "https://github.com/input-output-hk/cardano-node";
>     # A standard release compatible with the cardano-wallet commit is always preferred.
>     rev = "1.34.1";
>     sha256 = "1hh53whcj5y9kw4qpkiza7rmkniz18r493vv4dzl1a8r5fy3b2bv";
>   })
>   { };
>
>in
>  haskell.project.shellFor {
>    withHoogle = false;
>    
>    
>    nativeBuildInputs = with plutus-starter; [
>      hlint
>      cabal-install
>      cardano-node.cardano-cli
>      cardano-node.cardano-node
>      haskell-language-server
>      stylish-haskell
>      pkgs.niv
>      cardano-repo-tool
>      pkgs.ghcid
>      # HACK: This shouldn't need to be here.
>      pkgs.lzma.dev
>    ] ++ (pkgs.lib.optionals pure [
>      pkgs.git
>      pkgs.cacert
>      pkgs.curl
>      pkgs.jq
>    ]);
>  }
>```

## 4. Build the necessary package right ahead
* Run `nix-shell` in the terminal to build necessary packages first (expect quite a long build time here)

## 5. Change `cabal` setting to the project setting
* Delete the whole example folder coz it is not related to our projects.
* plutus-starter.cabal
> * Change the file name to others, for example, if my project name is sidex, change it to `sidex.cabal`
> * Delete anything below `library` to avoid `cabal repl` error of referring to other packages.
> * Amend the `library` to delete anything related to our project, example:
> ```
> library
>    import: lang
>    exposed-modules:
>      Sidex
>    build-depends:
>      base >= 4.9 && < 5,
>      aeson,
>      bytestring,
>      containers,
>      freer-extras,
>      playground-common,
>      plutus-contract,
>      plutus-tx-plugin,
>      plutus-tx,
>      plutus-ledger,
>    hs-source-dirs: src
> 
* cabal.project
> * Change the third line into your project name (e.g. here `packages: sidex.cabal`)

## 6. Happy Plutus Developing!

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

## 4. Copy a PPP cabal.project
* copy one cabal.project file to the directory with the `packages` on the top changed to the `.cabal` name of the project

## 5. cabal repl in terminal to compile
