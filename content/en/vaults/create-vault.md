---
title: Create a Vault
description: ''
position: 11
category: 'Vaults'
---

## Overview

The Yieldster Automation Platform provides an easy drag&drop UI to setup your vaults. Click on "Create Vault" in the main menu.

<img src="./screenshots/create-vault-overview.jpg" width="1280" height="941" alt="Create a vault"/>

## 1. Support

<img src="./screenshots/create-vault-support.png" width="540" height="390" alt="Create a vault"/>

Add deposit strategy, withdrawal strategy, allocation percentage for profit management fees, and the beneficiary address.

### Profit Management Fee

A vault admin can set a profit management fee ranging from 0% to 5% in the Allocation field. The profit management fee will be sent to the beneficiary address specified by the vault admin/creator.

<alert type="warning">

You cannot create a vault until you provide the beneficiary address.

</alert>

## 2. Assets

<img src="./screenshots/create-vault-assets.png" width="540" height="318" alt="Create a vault"/>

Drag and drop desired assets from the left sidebar into the asset field. Optional you can add vaults, too.

## 3. Core

<img src="./screenshots/create-vault-core.png" width="540" height="221" alt="Create a vault"/>

Select an advisor and customize the settings.

Read more about advisors [here](/advisors/advisors).

## 4. Admin

<img src="./screenshots/create-vault-admin.png" width="540" height="454" alt="Create a vault"/>

Enter the vault name, the name of the token minted by the vault, and the symbol of the token minted by the vault, all of which are unique to the vault. Also you can upload a vault icon.

The vault admin's wallet address is pre-populated.

### Emergency vault

A vault admin must add the Emergency Vault. Every vault has an emergency exit functionality, which is a function call that is to be triggered whenever the vault is compromised due to any technical fault. In such instances, the assets in the vault will be moved to this emergency vault address.

## Final Validation

When all of the cards in the Vault creation are complete, the vault administrator will be directed to validate five steps: vault proxy creation, register with APS, set vault assets, set token information, and set vault beneficiary. Each step must also be authorized in MetaMask.When this is verified, the vault is successfully created.

<alert type="warning">

You will be charged for all five transaction that will deducted from your wallet.

</alert>

<alert type="info">

Taking into account the verification and authorization of all five steps, the creation of a vault takes about 5 minutes.

</alert>


