# Yellow Network API Documentation

## Overview

Yellow Network is a Layer-3 peer-to-peer financial network that uses state channels technology to enable real-time cross-chain trading without intermediaries. This documentation covers all public APIs, functions, and components available for developers building on Yellow Network.

## Table of Contents

1. [Core Contracts](#core-contracts)
2. [State Channels (Nitro)](#state-channels-nitro)
3. [Clearing Protocol](#clearing-protocol)
4. [Token Contracts](#token-contracts)
5. [Utility Functions](#utility-functions)
6. [TypeScript SDK](#typescript-sdk)
7. [Network Deployment Information](#network-deployment-information)

---

## Core Contracts

### YellowToken

The core ERC20 token contract for both YELLOW and DUCKIES tokens.

#### Contract Address
- **Ethereum Mainnet**: `0x90b7E285ab6cf4e3A2487669dba3E339dB8a3320`
- **Polygon**: `0x18e73A5333984549484348A94f4D219f4faB7b81`

#### Public Functions

```solidity
function decimals() public pure override returns (uint8)
```
**Description**: Returns the number of decimals used for token representation.
**Returns**: `8` (overrides ERC20 default of 18)
**Example**:
```javascript
const decimals = await yellowToken.decimals(); // Returns 8
```

**Standard ERC20 Functions**:
- `balanceOf(address account)` - Get token balance
- `transfer(address to, uint256 amount)` - Transfer tokens
- `approve(address spender, uint256 amount)` - Approve spending
- `transferFrom(address from, address to, uint256 amount)` - Transfer from approved account

#### Usage Example

```javascript
// Using ethers.js
const yellowToken = new ethers.Contract(
  "0x90b7E285ab6cf4e3A2487669dba3E339dB8a3320",
  yellowTokenABI,
  provider
);

// Check balance
const balance = await yellowToken.balanceOf("0x...");
console.log(`Balance: ${ethers.utils.formatUnits(balance, 8)} YELLOW`);

// Transfer tokens
const tx = await yellowToken.transfer("0x...", ethers.utils.parseUnits("100", 8));
await tx.wait();
```

---

## State Channels (Nitro)

### NitroAdjudicator

The main contract for managing state channels on Yellow Network.

#### Key Functions

```solidity
function concludeAndTransferAllAssets(
    FixedPart memory fixedPart,
    SignedVariablePart memory candidate
) public virtual
```
**Description**: Finalizes a channel and liquidates all assets.
**Parameters**:
- `fixedPart`: Immutable channel properties
- `candidate`: Final state to conclude with

```solidity
function transferAllAssets(
    bytes32 channelId,
    Outcome.SingleAssetExit[] memory outcome,
    bytes32 stateHash
) public virtual
```
**Description**: Liquidates all assets for a finalized channel.
**Parameters**:
- `channelId`: Unique channel identifier
- `outcome`: Asset distribution array
- `stateHash`: Stored state hash for verification

#### State Channel Lifecycle

1. **Channel Opening**
   ```javascript
   // Create channel with initial state
   const channelId = await adjudicator.createChannel(
     participants,
     challengeDuration,
     initialState
   );
   ```

2. **State Updates**
   ```javascript
   // Update channel state off-chain
   const newState = createStateUpdate(channelId, turnNum, outcome);
   const signatures = await signState(newState, participants);
   ```

3. **Channel Closing**
   ```javascript
   // Conclude channel and transfer assets
   await adjudicator.concludeAndTransferAllAssets(fixedPart, finalState);
   ```

### ForceMove Protocol

Implements the core state channel protocol for dispute resolution.

#### Key Functions

```solidity
function challenge(
    FixedPart memory fixedPart,
    RecoveredVariablePart[] memory proof,
    RecoveredVariablePart memory candidate,
    RecoveredVariablePart memory challenger
) external returns (uint48)
```
**Description**: Submits a challenge to dispute the current channel state.

```solidity
function respond(
    bytes32 channelId,
    bool[] memory currentProof,
    RecoveredVariablePart memory response
) external returns (uint48)
```
**Description**: Responds to a challenge with a more recent state.

---

## Clearing Protocol

### MarginAppV1

Implements margin trading functionality within state channels.

#### Public Functions

```solidity
function stateIsSupported(
    FixedPart calldata fixedPart,
    RecoveredVariablePart[] calldata proof,
    RecoveredVariablePart calldata candidate
) external pure override returns (bool, string memory)
```
**Description**: Validates if a proposed state transition is valid.
**Parameters**:
- `fixedPart`: Channel configuration
- `proof`: Supporting proof states
- `candidate`: Proposed new state
**Returns**: Validity boolean and error message

#### Supported States

1. **Prefund** (turnNum = 0): Initial channel funding
2. **Postfund** (turnNum = 1): Channel is funded and ready
3. **Margin Call** (turnNum ≥ 2): Ongoing margin adjustments
4. **Final** (turnNum ≥ 3): Channel closure

#### Example Usage

```javascript
// Create margin trading channel
const marginChannel = {
  participants: [trader1, trader2],
  app: marginAppAddress,
  challengeDuration: 86400, // 24 hours
  outcome: [
    {
      asset: "0x...", // USDT address
      allocations: [
        { destination: trader1, amount: "1000000000" }, // $1000 USDT
        { destination: trader2, amount: "1000000000" }
      ]
    }
  ]
};
```

### EscrowApp

Implements secure escrow functionality for asset swaps.

#### Public Functions

```solidity
function stateIsSupported(
    FixedPart calldata fixedPart,
    RecoveredVariablePart[] calldata proof,
    RecoveredVariablePart calldata candidate
) external pure override returns (bool, string memory)
```

#### Supported States

1. **Prefund** (turnNum = 0): Initial setup
2. **Postfund** (turnNum = 1): Assets escrowed
3. **Settlement** (turnNum = 2): Asset swap executed
4. **Final** (turnNum = 3): Escrow completed

---

## Token Contracts

### DUCKIES Token

Canary network token for testing Yellow Network features.

#### Deployment Addresses
- **Polygon**: `0x18e73A5333984549484348A94f4D219f4faB7b81`
- **Ethereum**: `0x90b7E285ab6cf4e3A2487669dba3E339dB8a3320`

#### Key Features
- ERC20 compliant with 8 decimal places
- Used for testing and canary network operations
- Integrated with Ducklings NFT game

---

## Utility Functions

### Signature Utilities

```typescript
// src/nitro/signatures.ts

export function signState(
  state: State,
  privateKey: string
): Promise<string>
```
**Description**: Signs a state channel state with a private key.

```typescript
export function recoverSigner(
  message: string,
  signature: string
): string
```
**Description**: Recovers the signer address from a signature.

### Channel Utilities

```typescript
// src/nitro/channel.ts

export function getChannelId(
  participants: string[],
  appDefinition: string,
  challengeDuration: number
): string
```
**Description**: Generates a unique channel identifier.

### Transaction Utilities

```typescript
// src/nitro/transactions.ts

export function createAllocation(
  destination: string,
  amount: string
): Allocation
```
**Description**: Creates an allocation for outcome distribution.

---

## TypeScript SDK

### Installation

```bash
npm install @yellow-network/sdk
```

### Basic Usage

```typescript
import { YellowNetwork, StateChannel } from '@yellow-network/sdk';

// Initialize Yellow Network connection
const yellowNet = new YellowNetwork({
  provider: provider,
  networkId: 1, // Ethereum mainnet
});

// Create a trading channel
const channel = await yellowNet.createTradingChannel({
  counterparty: "0x...",
  asset: "0x...", // USDT
  amount: "1000000000", // $1000
  leverage: 5
});

// Execute a trade
const trade = await channel.executeTrade({
  side: "buy",
  amount: "0.1", // ETH
  price: "3000" // USDT per ETH
});
```

### Channel Management

```typescript
// Open a new channel
const channelData = await yellowNet.openChannel({
  participants: [myAddress, counterpartyAddress],
  assets: [usdtAddress],
  amounts: ["1000000000", "1000000000"]
});

// Get channel balance
const balance = await channel.getBalance(myAddress);

// Close channel
await channel.close();
```

### Event Listeners

```typescript
// Listen for channel events
channel.on('StateUpdate', (state) => {
  console.log('New state:', state);
});

channel.on('Challenge', (challenge) => {
  console.log('Channel challenged:', challenge);
});

channel.on('Concluded', (finalState) => {
  console.log('Channel concluded:', finalState);
});
```

---

## Network Deployment Information

### Mainnet Deployments

#### Ethereum Mainnet (Chain ID: 1)
- **YellowToken**: `0x90b7E285ab6cf4e3A2487669dba3E339dB8a3320`
- **DailyAirClaim**: `0x5df971419a39CC846B801b22D56Af59234b86238`
- **LiteVault**: `0xb5F3a9dD92270f55e55B7Ac7247639953538A261`

#### Polygon (Chain ID: 137)
- **YellowToken**: `0x18e73A5333984549484348A94f4D219f4faB7b81`
- **YellowAdjudicator**: `0xf81A43EBA92538B0323fCDb1A040F2183B352Ca3`
- **DucklingsV1 NFT**: `0x435b74f6DC4A0723CA19e4dD2AC8Aa1361c7B0f0`

#### Optimism (Chain ID: 10)
- **LiteVault**: `0xb5F3a9dD92270f55e55B7Ac7247639953538A261`
- **VoucherRouter**: `0x852b6eF2fBF26f25d1096CDd24bD6f5F9D62B301`

#### Base (Chain ID: 8453)
- **VoucherRouter**: `0x2A8B51821884CF9A7ea1A24C72E46Ff52dCb4F16`
- **LiteVault**: `0xb5F3a9dD92270f55e55B7Ac7247639953538A261`

### Testnet Deployments

#### Sepolia
- **YellowAdjudicator**: `0x47871f064d0b2ABf9190275C4D69f466C98fBD77`
- **MarginApp**: `0xa6F5563CD2D38a0c1F2D41DF7Eff7181bf3c6a7e`
- **EscrowApp**: `0xcccb67333fEefb04e85521fF0c219Cdb12539b84`

### Network Configuration

```typescript
const networks = {
  ethereum: {
    chainId: 1,
    rpcUrl: "https://eth.llamarpc.com",
    adjudicator: "0x...",
    token: "0x90b7E285ab6cf4e3A2487669dba3E339dB8a3320"
  },
  polygon: {
    chainId: 137,
    rpcUrl: "https://polygon.llamarpc.com",
    adjudicator: "0xf81A43EBA92538B0323fCDb1A040F2183B352Ca3",
    token: "0x18e73A5333984549484348A94f4D219f4faB7b81"
  },
  sepolia: {
    chainId: 11155111,
    rpcUrl: "https://sepolia.infura.io/v3/YOUR_KEY",
    adjudicator: "0x47871f064d0b2ABf9190275C4D69f466C98fBD77",
    token: "0x63FD175d3215779deBA7532fC660fA0E10c18676"
  }
};
```

---

## Error Handling

### Common Errors

```typescript
// Channel not found
if (!channel) {
  throw new Error("Channel does not exist");
}

// Insufficient balance
if (balance < amount) {
  throw new Error("Insufficient balance for transaction");
}

// Invalid signature
if (!isValidSignature(state, signature)) {
  throw new Error("Invalid state signature");
}
```

### Best Practices

1. **Always check channel status** before performing operations
2. **Validate signatures** before processing state updates
3. **Handle network errors** gracefully with retries
4. **Monitor gas prices** for optimal transaction timing
5. **Implement proper logging** for debugging

---

## Support and Resources

- **Documentation**: https://docs.yellow.org
- **GitHub**: https://github.com/layer-3/clearsync
- **Discord**: https://discord.gg/yellow
- **Telegram**: https://t.me/yellow_org
- **Whitepaper**: docs/whitepaper.md

For technical support or questions, please reach out through our community channels or create an issue on GitHub.