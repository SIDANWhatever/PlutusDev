# PlutusDev
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

## 3. Create 
