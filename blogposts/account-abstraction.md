With our community voting in full favor on the [IoTeX Improvement Proposal 14](https://community.iotex.io/t/iip-14-account-abstraction-via-entrypoint-contract/10339), Account Abstraction has finally hit the IoTeX Mainnet and Testnet, and its features are now available for all ecosystem devs. So, what is AA, how does it work, and how can you use it in your next application?

## A Quick Refresher 

Account Abstraction (AA) as defined by [ERC-4337](https://eips.ethereum.org/EIPS/eip-4337), "allows users to use smart contract wallets containing arbitrary verification logic instead of EOAs as their primary account." ERC-4337 introduces many user experience benefits, most notably enabling people to use Smart Contracts as their primary accounts.

ERC-4337 runs on top of the blockchain and does not require any changes to the blockchain itself. Currently, the IoTeX Account Abstraction code is based on ERC-4337 0.6.0 release version. 

## Components of the AA Infra

![AA](https://github.com/iotexproject/dev-portal-content/assets/77351244/712074ab-be62-4584-a998-1a322d75af13)

The components of the AA infrastructure are: 
- **Bundler Services**: one endpoint for **Mainnet** (https://bundler.w3bstream.com) and one for **Testnet** (https://bundler.testnet.w3bstream.com). A bundler is an offchain node that aggregates multiple abstracted user operations into a single transaction that the underlying blockchain can process. This transaction is sent to the other fixed component, called the `EntryPoint` Contract.
- `EntryPoint` Contract: There are two `EntryPoint` contracts deployed on IoTeX, one for **Mainnet** (`0xc3527348De07d591c9d567ce1998eFA2031B8675`) and one for **Testnet** (`0xc3527348De07d591c9d567ce1998eFA2031B8675`). An `EntryPoint` contract is in charge of creating/deploying certain special contracts, called `AccountFactory` contracts, which in turn are in charge of creating certain accounts (wallet contracts) that can be used for specific purposes.

In order to use account abstraction to create a new custom account, there are certain components that the dApp developer will have to create based on the needs of their application: 
- `Account` contract, which implements the validation logic in the `validateUserOp` method, and any execution logic that a user operation can require.
- `AccountFactory` contract, which is in charge, as said above, of creating/deploying new custom account contracts.
- Some client code that builds the user ops that are compatible with the verification rules implemented in the `AccountFactory`. 
- A **paymaster** is an optional part of the AA architecture. IoTeX provides a paymaster service for **Testnet** only at https://paymaster.testnet.w3bstream.com. The role of the paymaster is to sponsor the gas needed to execute user operations, either sponsoring them completely or allowing users to pay for them in various tokens.

## Example: The `P256AccountFactory`

As a first example, we have provided an official `P256AccountFactory` contract (Mainnet `0xD98d2B6cBca981c777037c5784721d8179D7030b` and Testnet `0x508Db1A73FcBA98594679aD4f5d8D0B880BbdaFB`) that allows developers to create account contracts that can verify user operations signed with the "p256" cryptography, rather than with Ethereum and IoTeX native "secp256k1" elliptic curve. This is incredibly useful, as it empowers developers to create applications where users can, for example, sign transactions with their biometrics, or move away from seed phrases, or even have superior security when their device supports a dedicated security chip (e.g. Android's Secure Element and Apple's Secure Enclave, etc.). The source code of the `P256AccountFactorycan` be found at https://github.com/iotexproject/account-abstraction-contracts/blob/main/contracts/accounts/secp256r1/P256AccountFactory.sol while the open source Account Abstraction contracts rely on the implementation by the original author of EIP-4337 for Ethereum here https://github.com/iotexproject/account-abstraction-contracts/tree/main.

The `P256AccountFactory` also supports the management of a paymaster service, which is made out of two components, a `VerifyingPaymaster` contract (https://github.com/iotexproject/account-abstraction-contracts/blob/main/contracts/paymaster/VerifyingPaymaster.sol) and an off-chain service endpoint to generate payment proof for the paymaster contract (https://paymaster.testnet.w3bstream.com, for Testnet only). 

The code below shows you how to interact with the p256 account implementation from a javascript client in order to [create an account](https://github.com/iotexproject/account-abstraction-contracts/blob/main/scripts/secp256r1/create.ts): 

```typescript
async function main() {
    // load deployed contracts
    const factory = (await ethers.getContract("P256AccountFactory")) as P256AccountFactory
    const entryPoint = (await ethers.getContract("EntryPoint")) as EntryPoint

    // an EOA account for send UserOperations
    const bundler = new ethers.Wallet(process.env.BUNDLER!, ethers.provider)

    // load secp256r1 keypair
    const keyContent = fs.readFileSync(path.join(__dirname, "key.pem"))
    const keyPair = ecPem.loadPrivateKey(keyContent)

    const publicKey = "0x" + keyPair.getPublicKey("hex").substring(2)
    const index = 0
    const account = await factory.getAddress(publicKey, index)

    // create create account UserOperation
    const initCode = hexConcat([
        factory.address,
        factory.interface.encodeFunctionData("createAccount", [publicKey, index]),
    ])
    const createOp = {
        sender: account,
        initCode: initCode,
    }

    const fullCreateOp = await fillUserOp(createOp, entryPoint)

    // stake IOTX for gas
    const stake = await entryPoint.balanceOf(account)
    if (stake.isZero()) {
        console.log(`deposit gas for account ${account}`)
        const tx = await entryPoint
            .connect(bundler)
            .depositTo(account, { value: ethers.utils.parseEther("10") })
        await tx.wait()
    }

    // sign UserOperation using secp256r1 curve
    const chainId = (await ethers.provider.getNetwork()).chainId
    const signedOp = await signOp(
        fullCreateOp,
        entryPoint.address,
        chainId,
        new P2565Signer(keyPair)
    )

    // simulate UserOperation
    const err = await entryPoint.callStatic.simulateValidation(signedOp).catch((e) => e)
    if (err.errorName === "FailedOp") {
        console.error(`simulate op error ${err.errorArgs.at(-1)}`)
        return
    } else if (err.errorName !== "ValidationResult") {
        console.error(`unknow error ${err}`)
        return
    }
    console.log(`simulate op success`)

    // send UserOpersion to EntryPoint
    const tx = await entryPoint.connect(bundler).handleOps([signedOp], bundler.address)
    console.log(`create account tx: ${tx.hash}, account: ${account}`)
}
```

While the following code will show you how to [transfer IOTX using bundler service and paymaster](https://github.com/iotexproject/account-abstraction-contracts/blob/main/scripts/secp256r1/transfer-bundler.ts): 

```typescript
async function main() {
    const factory = (await ethers.getContract("P256AccountFactory")) as P256AccountFactory
    const accountTpl = await ethers.getContractFactory("P256Account")
    const entryPoint = (await ethers.getContract("EntryPoint")) as EntryPoint
    const paymaster = await ethers.getContract("VerifyingPaymaster")
    const bundler = new JsonRpcProvider("http://localhost:4337")

    const signer = new ethers.Wallet(process.env.PRIVATE_KEY!)

    const keyContent = fs.readFileSync(path.join(__dirname, "key.pem"))
    const keyPair = ecPem.loadPrivateKey(keyContent)

    const publicKey = "0x" + keyPair.getPublicKey("hex").substring(2)

    const index = 0
    const account = await factory.getAddress(publicKey, index)

    const callData = accountTpl.interface.encodeFunctionData("execute", [
        "0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266",
        ethers.utils.parseEther("0.1"),
        "0x",
    ])

    const transferOp = {
        sender: account,
        callData,
        preVerificationGas: 50000,
    }

    const fullCreateOp = await fillUserOp(transferOp, entryPoint)
    fullCreateOp.paymasterAndData = hexConcat([
        paymaster.address,
        defaultAbiCoder.encode(["uint48", "uint48"], [0, 0]),
        "0x" + "00".repeat(65),
    ])

    const validAfter = Math.floor(new Date().getTime() / 1000)
    const validUntil = validAfter + 86400 // one day
    const pendingOpHash = await paymaster.getHash(fullCreateOp, validUntil, validAfter)
    const paymasterSignature = await signer.signMessage(arrayify(pendingOpHash))
    fullCreateOp.paymasterAndData = hexConcat([
        paymaster.address,
        defaultAbiCoder.encode(["uint48", "uint48"], [validUntil, validAfter]),
        paymasterSignature,
    ])

    const chainId = (await ethers.provider.getNetwork()).chainId
    const signedOp = await signOp(
        fullCreateOp,
        entryPoint.address,
        chainId,
        new P2565Signer(keyPair)
    )

    const err = await entryPoint.callStatic.simulateValidation(signedOp).catch((e) => e)
    if (err.errorName === "FailedOp") {
        console.error(`simulate op error ${err.errorArgs.at(-1)}`)
        return
    } else if (err.errorName !== "ValidationResult") {
        console.error(`unknow error ${err}`)
        return
    }
    console.log(`simulate op success`)

    const hexifiedUserOp = deepHexlify(await resolveProperties(signedOp))
    const result = await bundler.send("eth_sendUserOperation", [hexifiedUserOp, entryPoint.address])
    console.log(`transfer use bundler success opHash: ${result}`)
}
```

The rest of the example on how to interact with the **p256** account implementation from a javascript client, can be found at https://github.com/iotexproject/account-abstraction-contracts/tree/main/scripts/secp256r1






