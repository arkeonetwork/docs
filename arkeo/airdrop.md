# Arkeo Airdrop

## Introduction
The initial airdrop of arkeo will be spread amount a number of different groups from a broad variety of communites in order to foster participation in the Arkeo network. 

## Airdrop Distribution
Coming soon...

## Claiming your airdrop
Users are required to take multiple actions in order to recieve the full amount of their airdrop.

- 1/3 is sent to users when they claim their airdop
- 1/3 is send to users after they delegate arkeo tokens to a validator
- 1/3 is sent to users after they have voted in governance

Ethereum users will be able to claim on arkeo using a signed message that transfers their airdrop from the designated Ethereum address to their Arkeo address.

Addresses eligible for native claims on Arkeo, will have a small amount of Arkeo in their accounts on genesis. This will be enough to pay for the gas fees of claiming their initial airdrop.

To incentivize users to claim in a timely manner, the amount of claimable airdrop reduces over time. Users can claim the full airdrop amount for three months (`DurationUntilDecay`).
After three months, the claimable amount linearly decays until 6 months after launch. (At which point none of it is claimable) This is controlled by the parameter `DurationOfDecay` in the code, which is set to 3 months. (6 months - 3 months).

After 6 months from launch, all unclaimed airdrop tokens are sent back to the reserve.

## Thorchain users 
Due to the fact that thorchain has a unique bip44 coin type (931) compared with many other cosmos zones and Arkeo (118), we have added a mechanism to allow thorchain users to transfer their airdrop from the initially allocated address derived using the thorchain coin type to their preferred Arkeo address. This is only able to be done once per claim. 