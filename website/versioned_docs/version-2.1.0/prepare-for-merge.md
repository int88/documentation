---
id: prepare-for-merge
title: Prepare for The Merge
sidebar_label: Prepare for The Merge
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

:::caution DRAFT

**This document is being actively developed** and may change as we receive feedback from users like you. Join our [Discord server](https://discord.gg/prysmaticlabs) to share your feedback.

:::


[The Merge](https://ethereum.org/en/upgrades/merge/) will fully transition Ethereum's consensus mechanism from proof-of-work to proof-of-stake. This is made possible by the [Beacon Chain](https://ethereum.org/en/upgrades/beacon-chain/), a new Ethereum network layer that implements proof-of-stake consensus. After The Merge, this consensus layer will be fully "merged" with Ethereum's execution layer, and miners will be fully replaced by validators on Ethereum Mainnet.

This guide will walk you through the changes that you need to make in preparation for The Merge.

:::info New user?

This guidance is targeted at users who are already running Prysm. If you're starting from scratch, our [Quickstart](./install/install-with-script.md) will help you configure a Merge-ready node from start to finish.

:::

## The Merge: Before and after

| Before The Merge                                                                           | After The Merge                                                                                 |
|--------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------|
| You don't need to run a local execution client. You can use a service like Infura instead. | You **do** need to run an execution client. You **can't** use a service like Infura.            |
| Execution clients don't need to use the engine API.                                        | Execution clients **do** need to use the engine API.                                            |
| Execution clients don't need to connect to beacon node clients using JWT.                  | Execution clients **do** need to connect to beacon node clients using JWT.                      |
| Execution clients don't need to specify a TTD value.                                       | Execution clients **do** need to specify a TTD value.                                           |
| Miners receive tips.                                                                       | Validators receive tips.                                                                        |
| Beacon nodes don't need to use the engine API when connecting to execution nodes.          | Beacon nodes <strong>do</strong> need to use the engine API when connecting to execution nodes. |
| A 1TB hard drive is enough.                                                                | A 2TB+ SSD is required.                                                                         |


<br />


Let's step through each of these changes. 


## Generate JWT token

A secret **JWT token** will allow your beacon node to form an authenticated HTTP connection with your execution node's engine API. Although Ropsten and Sepolia are the only networks that currently require authenticated HTTP, it will soon be required on Goerli-Prater and Mainnet. We recommend doing this now regardless of the network you're running on.

Using [Prysm v2.1.3-rc.4](https://github.com/prysmaticlabs/prysm/releases/tag/v2.1.3-rc.4), issue the following command to generate this token:

<Tabs groupId="os" defaultValue="others" values={[
    {label: 'Windows', value: 'win'},
    {label: 'Linux, MacOS, Arm64', value: 'others'}
]}>
  <TabItem value="win">
  <pre><code>prysm.bat beacon-chain jwt generate-auth-secret</code></pre>
  </TabItem>
  <TabItem value="others">
  <pre><code>./prysm.sh beacon-chain jwt generate-jwt-secret</code></pre>
  </TabItem>
</Tabs>

Prysm will output a `jwt.hex` file path. If you're running on **Ropsten** or **Sepolia**, record this - we'll use it in the next step. If you're running on **Mainnet** or **Goerli-Prater**, you won't use this now, but be prepared to use when these networks are Merged.


## Configure execution node

Using the latest version of your execution client software, issue the following command to configure your execution node to consume your JWT token, and to expose an engine API endpoint: 

<Tabs groupId="execution-clients" defaultValue="nethermind" values={[
{label: 'Nethermind', value: 'nethermind'},
{label: 'Besu', value: 'besu'},
{label: 'Geth', value: 'geth'}
]}>
  <TabItem value="nethermind">
    <Tabs groupId="network" defaultValue="mainnet" values={[
        {label: 'Ropsten', value: 'ropsten'},
        {label: 'Sepolia', value: 'sepolia'},
        {label: 'Goerli-Prater', value: 'goerli-prater'},
        {label: 'Mainnet', value: 'mainnet'}
    ]}>
      <TabItem value="ropsten">
        <pre><code>Nethermind.Runner --config ropsten --JsonRpc.Enabled true --JsonRpc.JwtSecretFile=path/to/jwt.hex --Merge.TerminalTotalDifficulty 50000000000000000</code></pre>
      </TabItem>
       <TabItem value="sepolia">
        <pre><code>Nethermind.Runner --config sepolia --JsonRpc.Enabled true --JsonRpc.JwtSecretFile=path/to/jwt.hex --Merge.TerminalTotalDifficulty 50000000000000000</code></pre>
      </TabItem>
      <TabItem value="goerli-prater">
        <div class="admonition admonition-caution alert alert--warning">
          <div class="admonition-content"><p><strong>Goerli</strong> isn't ready for Merge configuration yet, so no changes are needed.</p></div>
        </div>
        <pre><code>Nethermind.Runner --config goerli --JsonRpc.Enabled true</code></pre>
      </TabItem>
      <TabItem value="mainnet">
        <div class="admonition admonition-caution alert alert--warning">
          <div class="admonition-content"><p><strong>Mainnet</strong> isn't ready for Merge configuration yet, so no changes are needed.</p></div>
        </div>
        <pre><code>Nethermind.Runner --config mainnet --JsonRpc.Enabled true</code></pre>
      </TabItem>
    </Tabs>
    <p>See Nethermind's <a href='https://docs.nethermind.io/nethermind/first-steps-with-nethermind/running-nethermind-post-merge'>Running Nethermind Post Merge</a> for more information.</p>
  </TabItem>
  <TabItem value="besu">
    <Tabs groupId="network" defaultValue="mainnet" values={[
        {label: 'Ropsten', value: 'ropsten'},
        {label: 'Sepolia', value: 'sepolia'},
        {label: 'Goerli-Prater', value: 'goerli-prater'},
        {label: 'Mainnet', value: 'mainnet'}
    ]}>
      <TabItem value="ropsten">
        <pre><code>besu --network=ropsten --rpc-http-enabled --engine-jwt-enabled=true --engine-jwt-secret=path/to/jwt.hex  --engine-host-allowlist="*" --override-genesis-config="terminalTotalDifficulty=50000000000000000"</code></pre>
      </TabItem>
      <TabItem value="sepolia">
        <pre><code>besu --network=sepolia --rpc-http-enabled --engine-jwt-enabled=true --engine-jwt-secret=path/to/jwt.hex  --engine-host-allowlist="*" --override-genesis-config="terminalTotalDifficulty=50000000000000000"</code></pre>
      </TabItem>
      <TabItem value="goerli-prater">
        <div class="admonition admonition-caution alert alert--warning">
          <div class="admonition-content"><p><strong>Goerli</strong> isn't ready for Merge configuration yet, so no changes are needed.</p></div>
        </div>
        <pre><code>besu --network=goerli --rpc-http-enabled</code></pre>
      </TabItem>
      <TabItem value="mainnet">
        <div class="admonition admonition-caution alert alert--warning">
          <div class="admonition-content"><p><strong>Mainnet</strong> isn't ready for Merge configuration yet, so no changes are needed.</p></div>
        </div>
        <pre><code>besu --network=mainnet --rpc-http-enabled</code></pre>
      </TabItem>
    </Tabs>
    <p>See Besu's <a href='https://besu.hyperledger.org/en/stable/Reference/CLI/CLI-Syntax/'>command-line options</a> for parameter definitions.</p>
  </TabItem>
  <TabItem value="geth">
    <div class="admonition admonition-caution alert alert--warning">
      <div class="admonition-content"><p><strong>Geth is a supermajority execution-layer client</strong>. This centralization poses an active risk to the security of Ethereum. Consider using a minority execution-layer client to distribute this risk for the ecosystem <a class='footnote' href='#footnote-10'>[10]</a>.</p></div>
    </div>
    <Tabs groupId="network" defaultValue="mainnet" values={[
        {label: 'Ropsten', value: 'ropsten'},
        {label: 'Sepolia', value: 'sepolia'},
        {label: 'Goerli-Prater', value: 'goerli-prater'},
        {label: 'Mainnet', value: 'mainnet'}
    ]}>
      <TabItem value="ropsten">
        <pre><code>geth --ropsten --http --http.api eth,net,engine,admin --authrpc.vhosts="localhost" --authrpc.jwtsecret=path/to/jwt.hex --override.terminaltotaldifficulty 50000000000000000</code></pre>
      </TabItem>
      <TabItem value="sepolia">
        <pre><code>geth --sepolia --http --http.api eth,net,engine,admin --authrpc.vhosts="localhost" --authrpc.jwtsecret=path/to/jwt.hex --override.terminaltotaldifficulty 50000000000000000</code></pre>
      </TabItem>
      <TabItem value="goerli-prater">
        <div class="admonition admonition-caution alert alert--warning">
          <div class="admonition-content"><p><strong>Goerli</strong> isn't ready for Merge configuration yet, so no changes are needed.</p></div>
        </div>
        <pre><code>geth --goerli --http --http.api eth,net,engine,admin</code></pre>
      </TabItem>
      <TabItem value="mainnet">
        <div class="admonition admonition-caution alert alert--warning">
          <div class="admonition-content"><p><strong>Mainnet</strong> isn't ready for Merge configuration yet, so no changes are needed.</p></div>
        </div>
        <pre><code>geth --mainnet --http --http.api eth,net,engine,admin</code></pre>
      </TabItem>
    </Tabs>
    <p>See Geth's <a href='https://geth.ethereum.org/docs/interface/command-line-options'>command-line options</a> for parameter definitions.</p>
  </TabItem>
</Tabs>


## Configure beacon node

Next, we'll configure your beacon node to consume your JWT token so it can securely connect to your execution node's engine API endpoint. If you're running a validator, specifying a `suggested-fee-recipient` wallet address will allow you to earn what were previously miner tips:

<Tabs groupId="os" defaultValue="others" values={[
    {label: 'Windows', value: 'win'},
    {label: 'Linux, MacOS, Arm64', value: 'others'}
]}>
  <TabItem value="win">
    <Tabs groupId="network" defaultValue="mainnet" values={[
        {label: 'Ropsten', value: 'ropsten'},
        {label: 'Sepolia', value: 'sepolia'},
        {label: 'Goerli-Prater', value: 'goerli-prater'},
        {label: 'Mainnet', value: 'mainnet'}
    ]}>
      <TabItem value="ropsten">
        <p>Download the <a href='https://github.com/eth-clients/merge-testnets/blob/main/ropsten-beacon-chain/genesis.ssz'>Ropsten genesis state from Github</a>.</p>
        <pre><code>prysm.bat beacon-chain --http-web3provider=http://localhost:8551 --ropsten --jwt-secret=path/to/jwt.hex --genesis-state=genesis.ssz --suggested-fee-recipient=0x01234567722E6b0000012BFEBf6177F1D2e9758D9</code></pre>
      </TabItem>
      <TabItem value="sepolia">
      <p>Download the <a href='https://github.com/eth-clients/merge-testnets/blob/main/sepolia/genesis.ssz'>Sepolia genesis state from Github</a>.</p>
        <pre><code>prysm.bat beacon-chain --http-web3provider=http://localhost:8551 --sepolia --jwt-secret=path/to/jwt.hex --genesis-state=genesis.ssz --suggested-fee-recipient=0x01234567722E6b0000012BFEBf6177F1D2e9758D9</code></pre>
      </TabItem>
      <TabItem value="goerli-prater">
        <div class="admonition admonition-caution alert alert--warning">
          <div class="admonition-content"><p><strong>Prater</strong> isn't ready for Merge configuration yet, so no changes are needed.</p></div>
        </div>
        <p>Download the <a href='https://github.com/eth-clients/eth2-networks/raw/master/shared/prater/genesis.ssz'>Prater genesis state from Github</a>.</p>
        <pre><code>prysm.bat beacon-chain --http-web3provider=http://localhost:8545 --prater --genesis-state=genesis.ssz --suggested-fee-recipient=0x01234567722E6b0000012BFEBf6177F1D2e9758D9</code></pre>
      </TabItem>
      <TabItem value="mainnet">
        <div class="admonition admonition-caution alert alert--warning">
          <div class="admonition-content"><p><strong>Mainnet</strong> isn't ready for Merge configuration yet, so no changes are needed.</p></div>
        </div>
        <pre><code>prysm.bat beacon-chain --http-web3provider=http://localhost:8545 --mainnet --suggested-fee-recipient=0x01234567722E6b0000012BFEBf6177F1D2e9758D9</code></pre>
      </TabItem>
    </Tabs>
  </TabItem>
  <TabItem value="others">
    <Tabs groupId="network" defaultValue="mainnet" values={[
        {label: 'Ropsten', value: 'ropsten'},
        {label: 'Sepolia', value: 'sepolia'},
        {label: 'Goerli-Prater', value: 'goerli-prater'},
        {label: 'Mainnet', value: 'mainnet'}
    ]}> 
    <TabItem value="ropsten">
      <p>Download the <a href='https://github.com/eth-clients/merge-testnets/blob/main/ropsten-beacon-chain/genesis.ssz'>Ropsten genesis state from Github</a>.</p>
      <pre><code>./prysm.sh beacon-chain --http-web3provider=http://localhost:8551 --ropsten --jwt-secret=path/to/jwt.hex --genesis-state=genesis.ssz --suggested-fee-recipient=0x01234567722E6b0000012BFEBf6177F1D2e9758D9</code></pre>
    </TabItem>
    <TabItem value="sepolia">
      <p>Download the <a href='https://github.com/eth-clients/merge-testnets/blob/main/sepolia/genesis.ssz'>Sepolia genesis state from Github</a>.</p>
      <pre><code>./prysm.sh beacon-chain --http-web3provider=http://localhost:8551 --sepolia --jwt-secret=path/to/jwt.hex --genesis-state=genesis.ssz --suggested-fee-recipient=0x01234567722E6b0000012BFEBf6177F1D2e9758D9</code></pre>
    </TabItem>
    <TabItem value="goerli-prater">
      <div class="admonition admonition-caution alert alert--warning">
          <div class="admonition-content"><p><strong>Prater</strong> isn't ready for Merge configuration yet, so no changes are needed.</p></div>
      </div>
      <p>Download the <a href='https://github.com/eth-clients/eth2-networks/raw/master/shared/prater/genesis.ssz'>Prater genesis state from Github</a>.</p>
      <pre><code>./prysm.sh beacon-chain --http-web3provider=http://localhost:8545 --prater --genesis-state=genesis.ssz --suggested-fee-recipient=0x01234567722E6b0000012BFEBf6177F1D2e9758D9</code></pre>
    </TabItem>
    <TabItem value="mainnet">
      <div class="admonition admonition-caution alert alert--warning">
          <div class="admonition-content"><p><strong>Mainnet</strong> isn't ready for Merge configuration yet, so no changes are needed.</p></div>
      </div>
      <pre><code>./prysm.sh beacon-chain --http-web3provider=http://localhost:8545 --mainnet --suggested-fee-recipient=0x01234567722E6b0000012BFEBf6177F1D2e9758D9</code></pre>
    </TabItem>
  </Tabs>
  </TabItem>
</Tabs>


## Configure validator node

Validator client configuration doesn't need to be updated for The Merge. Fee recipient can optionally be configured on your validator node if you want redundancy or multiple fee recipient addresses. See [Configuring a Fee Recipient Address](./execution-node/fee-recipient.md) to learn more.


## Upgrade hardware

We recommend updating your hard drive to a 2TB+ SSD as soon as possible.

:::tip Congratulations!

You’re now ready for The Merge. If you have any questions, feel free ask them on our [Discord](https://discord.gg/prysmaticlabs).

:::


## Frequently asked questions

**I'm currently running a validator on Ethereum Mainnet. When should I make changes?**

You can make the above changes now. This document will be continuously updated with the latest recommended configuration.

**Can I use IPC post-Merge?**

Yes. You also won't have to worry about JWT if you use IPC. Refer to your clients' command-line documentation for IPC configuration parameters.