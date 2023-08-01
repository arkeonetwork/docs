# Welcome to Phase 3 of Arkeo Network's Testnet! 🚀

We are delighted to invite you to take part in an exciting journey as we
embark on Phase 3 of our Testnet. This crucial stage is a testament to our
relentless pursuit of developing a robust, efficient, and user-centric
protocol, and we couldn't have done it without community members like you. 

This document is your comprehensive guide to the exciting world of the Arkeo
public testnet, where your valuable contributions can make a significant
impact. Let's get started!

## 🎯 Purpose and Goals of the Testnet The Testnet serves as a vibrant
platform to test the fundamental functionality of our core protocol. This
includes validating network operations, data providers, and dapps that utilize
the data. However, note that we're saving the airdrop for a separate testnet,
specifically created to test that code. 

## ⏳ Testnet Duration Our Testnet is designed to operate for at least one
month, but the joy of discovery may extend this period. Keep in mind, we might
reset and launch a new network if our dedicated team deems it necessary for
optimization purposes.

## 🤔 Need Support or Have Questions?  Our supportive community is always
ready to help! If you have questions, concerns, creative ideas, or bug
reports, please join our [Discord server](https://discord.gg/wBcdVM53). For
more focused discussions, #validator-testing and #data-testing are your go-to
channels.

## 🤝 Expected Conduct We kindly ask all testers to contribute in good faith,
keeping the best interests of the network and the project at heart. If you
stumble upon a bug or discover an exploit during your exploration, please
alert our developers via Discord. 

## 🎩 Testing Roles There are three primary roles you can take for a spin in
our beta:

1. **Validator:** Help the network commit new blocks to the Arkeo blockchain
   by running a testnet validator.
2. **Data Provider:** Run full nodes of various blockchains, allowing others
   to contract your infrastructure and pay for access (or offer it for free).
3. **Dapps:** If you're developing applications that require connections to
   blockchain daemons, we'd love your input on this role.

Whatever your role, you should find what you need to get started in our
documentation: https://docs.arkeo.network - but if you can’t find what you are
looking for please reach out in discord and let us know how we can help.

## 💰 Acquiring Testnet Tokens To jump into testing, you'll need testnet
tokens. At present, we support a command-line tool, not wallets. Learn how to
install this CLI tool and create a wallet on our [GitHub
repository](https://github.com/arkeonetwork/arkeo#arkeo-binary). Once you've
got your "tarkeo" address, request tokens through this [Google
form](https://forms.gle/aM6sc73qtxenRxf37).

## 💻 Testnet Address
To access a testnet server, they can be reached at
```
seed.arkeo.network
```

The arkeo api is available on port `1317`

example
``
curl seed.arkeo.network:1317/arkeo/providers | jq
```

For the RPC port, try port `26657`
example
```
curl -s seed.arkeo.network:26657/status | jq
```

## 🎉 Let's Make Magic Happen!  By participating in the Arkeo Network Phase 3
Testnet, you're not just testing a protocol—you're part of a passionate
community that is shaping the future of blockchain technology. Your
contributions are helping us build a more robust, efficient, and user-centric
platform. For this, we are immensely grateful. Let the testing begin!
