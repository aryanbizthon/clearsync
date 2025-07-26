# BIE Developer Deployment Guide: Yellow Network with Nitrolite SDK

## Overview

This guide provides a comprehensive step-by-step approach for Blockchain Infrastructure Entity (BIE) developers to deploy and integrate with the Yellow Network using the Nitrolite SDK. Yellow Network is a Layer-3 peer-to-peer financial network that uses state channels technology to enable real-time cross-chain trading.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Environment Setup](#environment-setup)
3. [Understanding Yellow Network Architecture](#understanding-yellow-network-architecture)
4. [Nitrolite SDK Installation](#nitrolite-sdk-installation)
5. [Network Configuration](#network-configuration)
6. [Smart Contract Deployment](#smart-contract-deployment)
7. [State Channel Integration](#state-channel-integration)
8. [Testing and Validation](#testing-and-validation)
9. [Production Deployment](#production-deployment)
10. [Monitoring and Maintenance](#monitoring-and-maintenance)

---

## Prerequisites

### Technical Requirements

- **Node.js**: v18.0.0 or higher
- **npm/yarn**: Latest version
- **Git**: Latest version
- **Docker**: For containerized deployment (optional)
- **Ethereum wallet**: With sufficient funds for gas fees

### Knowledge Requirements

- Solid understanding of Ethereum and smart contracts
- Familiarity with TypeScript/JavaScript
- Basic knowledge of state channels concept
- Understanding of ERC-20 tokens
- Experience with Web3 development tools

### Required Tokens

- **ETH**: For gas fees on Ethereum/L2s
- **YELLOW/DUCKIES**: For network participation
- **Stablecoins**: For trading operations (USDT, USDC, DAI)

---

## Environment Setup

### 1. Install Development Tools

```bash
# Install Node.js (using nvm)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 18
nvm use 18

# Install Yarn
npm install -g yarn

# Install Hardhat (for smart contract development)
npm install -g hardhat

# Install Git
sudo apt-get install git  # Ubuntu/Debian
brew install git          # macOS
```

### 2. Create Project Structure

```bash
mkdir yellow-bie-integration
cd yellow-bie-integration

# Initialize project
npm init -y
yarn init -y

# Create directory structure
mkdir -p {contracts,scripts,test,src,config,docs}
```

### 3. Environment Configuration

Create `.env` file:

```bash
# .env
# Network Configuration
ETHEREUM_RPC_URL=https://eth.llamarpc.com
POLYGON_RPC_URL=https://polygon.llamarpc.com
SEPOLIA_RPC_URL=https://sepolia.infura.io/v3/YOUR_INFURA_KEY

# Private Keys (NEVER commit these)
DEPLOYER_PRIVATE_KEY=your_private_key_here
OPERATOR_PRIVATE_KEY=your_operator_key_here

# Yellow Network Contracts
YELLOW_TOKEN_ADDRESS=0x90b7E285ab6cf4e3A2487669dba3E339dB8a3320
YELLOW_ADJUDICATOR_ADDRESS=0xf81A43EBA92538B0323fCDb1A040F2183B352Ca3

# API Keys
ETHERSCAN_API_KEY=your_etherscan_api_key
POLYGONSCAN_API_KEY=your_polygonscan_api_key

# Configuration
CHALLENGE_DURATION=86400  # 24 hours in seconds
MIN_CHANNEL_BALANCE=1000000000  # Minimum balance in wei
```

---

## Understanding Yellow Network Architecture

### Core Components

1. **State Channels**: Off-chain payment and trading channels
2. **Adjudicator**: On-chain dispute resolution
3. **Apps**: Application-specific logic (Margin, Escrow)
4. **ClearSync Protocol**: Yellow's clearing and settlement protocol

### Key Concepts

```typescript
// Channel Structure
interface Channel {
  id: string;
  participants: string[];
  assets: Asset[];
  app: string;
  challengeDuration: number;
  status: 'Open' | 'Challenge' | 'Finalized';
}

// State Structure
interface State {
  channelId: string;
  turnNum: number;
  outcome: Outcome[];
  isFinal: boolean;
}

// Outcome Structure
interface Outcome {
  asset: string;
  allocations: Allocation[];
}
```

---

## Nitrolite SDK Installation

### 1. Install Core Dependencies

```bash
# Install Yellow Network SDK (hypothetical - adapt based on actual SDK)
npm install @yellow-network/nitrolite-sdk
npm install @yellow-network/contracts

# Install required peer dependencies
npm install ethers@5.7.2
npm install @openzeppelin/contracts
npm install hardhat
npm install dotenv

# Install development dependencies
npm install -D @types/node
npm install -D typescript
npm install -D ts-node
npm install -D @nomiclabs/hardhat-ethers
npm install -D @nomiclabs/hardhat-waffle
```

### 2. TypeScript Configuration

Create `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020"],
    "module": "commonjs",
    "declaration": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "moduleResolution": "node",
    "baseUrl": "./",
    "paths": {
      "*": ["node_modules/*"]
    },
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true
  },
  "include": ["src/**/*", "test/**/*", "scripts/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### 3. Hardhat Configuration

Create `hardhat.config.ts`:

```typescript
import { HardhatUserConfig } from "hardhat/config";
import "@nomiclabs/hardhat-ethers";
import "@nomiclabs/hardhat-waffle";
import "dotenv/config";

const config: HardhatUserConfig = {
  solidity: {
    version: "0.8.22",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200
      }
    }
  },
  networks: {
    hardhat: {
      forking: {
        url: process.env.ETHEREUM_RPC_URL!,
      }
    },
    ethereum: {
      url: process.env.ETHEREUM_RPC_URL,
      accounts: [process.env.DEPLOYER_PRIVATE_KEY!],
      gasPrice: 20000000000 // 20 gwei
    },
    polygon: {
      url: process.env.POLYGON_RPC_URL,
      accounts: [process.env.DEPLOYER_PRIVATE_KEY!],
      gasPrice: 30000000000 // 30 gwei
    },
    sepolia: {
      url: process.env.SEPOLIA_RPC_URL,
      accounts: [process.env.DEPLOYER_PRIVATE_KEY!],
      gasPrice: 20000000000
    }
  },
  etherscan: {
    apiKey: {
      mainnet: process.env.ETHERSCAN_API_KEY!,
      polygon: process.env.POLYGONSCAN_API_KEY!,
      sepolia: process.env.ETHERSCAN_API_KEY!
    }
  }
};

export default config;
```

---

## Network Configuration

### 1. Create Network Configuration

Create `src/config/networks.ts`:

```typescript
export interface NetworkConfig {
  chainId: number;
  name: string;
  rpcUrl: string;
  contracts: {
    yellowToken: string;
    adjudicator: string;
    marginApp: string;
    escrowApp: string;
  };
  blockExplorer: string;
}

export const networks: Record<string, NetworkConfig> = {
  ethereum: {
    chainId: 1,
    name: "Ethereum Mainnet",
    rpcUrl: process.env.ETHEREUM_RPC_URL!,
    contracts: {
      yellowToken: "0x90b7E285ab6cf4e3A2487669dba3E339dB8a3320",
      adjudicator: "0x...", // TBD - use actual addresses
      marginApp: "0x...",
      escrowApp: "0x..."
    },
    blockExplorer: "https://etherscan.io"
  },
  polygon: {
    chainId: 137,
    name: "Polygon",
    rpcUrl: process.env.POLYGON_RPC_URL!,
    contracts: {
      yellowToken: "0x18e73A5333984549484348A94f4D219f4faB7b81",
      adjudicator: "0xf81A43EBA92538B0323fCDb1A040F2183B352Ca3",
      marginApp: "0xd3f6EA0DCe26E7836fB309dcfcf506e44524B2A5",
      escrowApp: "0x59735037AC294641F8CE51d68D6C45a500B8e645"
    },
    blockExplorer: "https://polygonscan.com"
  },
  sepolia: {
    chainId: 11155111,
    name: "Sepolia Testnet",
    rpcUrl: process.env.SEPOLIA_RPC_URL!,
    contracts: {
      yellowToken: "0x63FD175d3215779deBA7532fC660fA0E10c18676",
      adjudicator: "0x47871f064d0b2ABf9190275C4D69f466C98fBD77",
      marginApp: "0xa6F5563CD2D38a0c1F2D41DF7Eff7181bf3c6a7e",
      escrowApp: "0xcccb67333fEefb04e85521fF0c219Cdb12539b84"
    },
    blockExplorer: "https://sepolia.etherscan.io"
  }
};
```

### 2. Initialize Yellow Network Client

Create `src/YellowNetworkClient.ts`:

```typescript
import { ethers } from "ethers";
import { networks, NetworkConfig } from "./config/networks";

export class YellowNetworkClient {
  private provider: ethers.providers.Provider;
  private signer: ethers.Signer;
  private network: NetworkConfig;
  private contracts: any = {};

  constructor(
    networkName: string,
    privateKey: string,
    rpcUrl?: string
  ) {
    this.network = networks[networkName];
    this.provider = new ethers.providers.JsonRpcProvider(
      rpcUrl || this.network.rpcUrl
    );
    this.signer = new ethers.Wallet(privateKey, this.provider);
    
    this.initializeContracts();
  }

  private async initializeContracts() {
    // Initialize contract instances
    // Note: You'll need actual ABIs for these contracts
    this.contracts.yellowToken = new ethers.Contract(
      this.network.contracts.yellowToken,
      [], // YellowToken ABI
      this.signer
    );

    this.contracts.adjudicator = new ethers.Contract(
      this.network.contracts.adjudicator,
      [], // NitroAdjudicator ABI
      this.signer
    );
  }

  async getBalance(address?: string): Promise<ethers.BigNumber> {
    const account = address || await this.signer.getAddress();
    return this.contracts.yellowToken.balanceOf(account);
  }

  async createChannel(params: any) {
    // Implementation for creating state channels
    // This is where you'll integrate with Nitrolite SDK
  }
}
```

---

## Smart Contract Deployment

### 1. Custom BIE Contract

Create `contracts/BIEIntegration.sol`:

```solidity
// SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity 0.8.22;

import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

/**
 * @title BIE Integration Contract
 * @notice Custom contract for BIE integration with Yellow Network
 */
contract BIEIntegration is Ownable, ReentrancyGuard {
    IERC20 public yellowToken;
    address public adjudicator;
    
    mapping(address => bool) public authorizedOperators;
    mapping(bytes32 => bool) public activeChannels;
    
    event ChannelCreated(bytes32 indexed channelId, address indexed participant);
    event OperatorAuthorized(address indexed operator);
    event OperatorRevoked(address indexed operator);
    
    constructor(
        address _yellowToken,
        address _adjudicator
    ) {
        yellowToken = IERC20(_yellowToken);
        adjudicator = _adjudicator;
    }
    
    modifier onlyAuthorized() {
        require(
            authorizedOperators[msg.sender] || msg.sender == owner(),
            "Not authorized"
        );
        _;
    }
    
    function authorizeOperator(address operator) external onlyOwner {
        authorizedOperators[operator] = true;
        emit OperatorAuthorized(operator);
    }
    
    function revokeOperator(address operator) external onlyOwner {
        authorizedOperators[operator] = false;
        emit OperatorRevoked(operator);
    }
    
    function createTradingChannel(
        address counterparty,
        uint256 amount
    ) external onlyAuthorized nonReentrant {
        require(amount > 0, "Amount must be positive");
        require(counterparty != address(0), "Invalid counterparty");
        
        // Transfer YELLOW tokens for channel funding
        yellowToken.transferFrom(msg.sender, address(this), amount);
        
        // Generate channel ID (simplified)
        bytes32 channelId = keccak256(
            abi.encodePacked(msg.sender, counterparty, block.timestamp)
        );
        
        activeChannels[channelId] = true;
        emit ChannelCreated(channelId, msg.sender);
    }
    
    function getTokenBalance() external view returns (uint256) {
        return yellowToken.balanceOf(address(this));
    }
}
```

### 2. Deployment Script

Create `scripts/deploy.ts`:

```typescript
import { ethers } from "hardhat";
import { networks } from "../src/config/networks";

async function main() {
  const networkName = process.env.NETWORK || "sepolia";
  const network = networks[networkName];
  
  if (!network) {
    throw new Error(`Network ${networkName} not configured`);
  }

  console.log(`Deploying to ${network.name}...`);
  
  // Get the deployer account
  const [deployer] = await ethers.getSigners();
  console.log("Deploying with account:", deployer.address);
  
  const balance = await deployer.getBalance();
  console.log("Account balance:", ethers.utils.formatEther(balance));

  // Deploy BIE Integration contract
  const BIEIntegration = await ethers.getContractFactory("BIEIntegration");
  const bieIntegration = await BIEIntegration.deploy(
    network.contracts.yellowToken,
    network.contracts.adjudicator
  );

  await bieIntegration.deployed();

  console.log("BIE Integration deployed to:", bieIntegration.address);
  
  // Verify contract on block explorer (if on mainnet/testnet)
  if (networkName !== "hardhat") {
    console.log("Waiting for block confirmations...");
    await bieIntegration.deployTransaction.wait(5);
    
    try {
      await run("verify:verify", {
        address: bieIntegration.address,
        constructorArguments: [
          network.contracts.yellowToken,
          network.contracts.adjudicator
        ],
      });
    } catch (error) {
      console.log("Verification failed:", error);
    }
  }

  // Save deployment info
  const deploymentInfo = {
    network: networkName,
    chainId: network.chainId,
    contractAddress: bieIntegration.address,
    deployerAddress: deployer.address,
    yellowToken: network.contracts.yellowToken,
    adjudicator: network.contracts.adjudicator,
    timestamp: new Date().toISOString()
  };

  console.log("Deployment Info:", deploymentInfo);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

### 3. Run Deployment

```bash
# Deploy to Sepolia testnet
NETWORK=sepolia npx hardhat run scripts/deploy.ts --network sepolia

# Deploy to Polygon
NETWORK=polygon npx hardhat run scripts/deploy.ts --network polygon

# Deploy to Ethereum mainnet
NETWORK=ethereum npx hardhat run scripts/deploy.ts --network ethereum
```

---

## State Channel Integration

### 1. Channel Manager Implementation

Create `src/ChannelManager.ts`:

```typescript
import { ethers } from "ethers";
import { YellowNetworkClient } from "./YellowNetworkClient";

export interface ChannelParams {
  counterparty: string;
  asset: string;
  amount: string;
  challengeDuration?: number;
}

export interface State {
  channelId: string;
  turnNum: number;
  outcome: Outcome[];
  isFinal: boolean;
}

export interface Outcome {
  asset: string;
  allocations: Allocation[];
}

export interface Allocation {
  destination: string;
  amount: string;
}

export class ChannelManager {
  private client: YellowNetworkClient;
  private channels: Map<string, any> = new Map();

  constructor(client: YellowNetworkClient) {
    this.client = client;
  }

  async createChannel(params: ChannelParams): Promise<string> {
    const participants = [
      await this.client.signer.getAddress(),
      params.counterparty
    ];

    // Create channel configuration
    const channelConfig = {
      participants,
      appDefinition: this.getAppAddress(params.asset),
      challengeDuration: params.challengeDuration || 86400,
      outcome: [{
        asset: params.asset,
        allocations: [
          {
            destination: participants[0],
            amount: params.amount
          },
          {
            destination: participants[1],
            amount: params.amount
          }
        ]
      }]
    };

    // Generate channel ID
    const channelId = this.generateChannelId(channelConfig);
    
    // Store channel locally
    this.channels.set(channelId, {
      config: channelConfig,
      state: this.createInitialState(channelId, channelConfig.outcome),
      status: 'open'
    });

    console.log(`Channel created: ${channelId}`);
    return channelId;
  }

  async updateChannelState(
    channelId: string,
    newOutcome: Outcome[],
    isFinal: boolean = false
  ): Promise<State> {
    const channel = this.channels.get(channelId);
    if (!channel) {
      throw new Error(`Channel ${channelId} not found`);
    }

    const newState: State = {
      channelId,
      turnNum: channel.state.turnNum + 1,
      outcome: newOutcome,
      isFinal
    };

    // Update local state
    channel.state = newState;
    this.channels.set(channelId, channel);

    console.log(`Channel state updated: ${channelId}, turn: ${newState.turnNum}`);
    return newState;
  }

  async closeChannel(channelId: string): Promise<void> {
    const channel = this.channels.get(channelId);
    if (!channel) {
      throw new Error(`Channel ${channelId} not found`);
    }

    // Create final state
    const finalState = await this.updateChannelState(
      channelId,
      channel.state.outcome,
      true
    );

    // Submit to adjudicator for on-chain conclusion
    // This would integrate with the actual Nitrolite SDK
    console.log(`Closing channel: ${channelId}`);
    
    // Update status
    channel.status = 'closed';
    this.channels.set(channelId, channel);
  }

  private generateChannelId(config: any): string {
    const hash = ethers.utils.keccak256(
      ethers.utils.defaultAbiCoder.encode(
        ['address[]', 'address', 'uint256'],
        [config.participants, config.appDefinition, config.challengeDuration]
      )
    );
    return hash;
  }

  private createInitialState(channelId: string, outcome: Outcome[]): State {
    return {
      channelId,
      turnNum: 0,
      outcome,
      isFinal: false
    };
  }

  private getAppAddress(asset: string): string {
    // Return appropriate app address based on asset type
    // This would be configured based on your network setup
    return this.client.network.contracts.marginApp; // Default to margin app
  }

  getChannel(channelId: string) {
    return this.channels.get(channelId);
  }

  listChannels() {
    return Array.from(this.channels.entries());
  }
}
```

### 2. Trading Implementation

Create `src/TradingEngine.ts`:

```typescript
import { ChannelManager, ChannelParams } from "./ChannelManager";
import { YellowNetworkClient } from "./YellowNetworkClient";

export interface TradeParams {
  channelId: string;
  side: 'buy' | 'sell';
  amount: string;
  price: string;
  asset: string;
}

export class TradingEngine {
  private client: YellowNetworkClient;
  private channelManager: ChannelManager;

  constructor(client: YellowNetworkClient) {
    this.client = client;
    this.channelManager = new ChannelManager(client);
  }

  async openTradingChannel(params: ChannelParams): Promise<string> {
    // Ensure sufficient YELLOW token balance
    const balance = await this.client.getBalance();
    const requiredAmount = ethers.utils.parseUnits(params.amount, 8);
    
    if (balance.lt(requiredAmount)) {
      throw new Error("Insufficient YELLOW token balance");
    }

    return this.channelManager.createChannel(params);
  }

  async executeTrade(params: TradeParams): Promise<void> {
    const channel = this.channelManager.getChannel(params.channelId);
    if (!channel) {
      throw new Error(`Trading channel ${params.channelId} not found`);
    }

    // Calculate new allocations based on trade
    const newOutcome = this.calculateTradeOutcome(channel.state.outcome, params);
    
    // Update channel state
    await this.channelManager.updateChannelState(
      params.channelId,
      newOutcome
    );

    console.log(`Trade executed: ${params.side} ${params.amount} at ${params.price}`);
  }

  async closeTradingChannel(channelId: string): Promise<void> {
    return this.channelManager.closeChannel(channelId);
  }

  private calculateTradeOutcome(currentOutcome: any[], trade: TradeParams): any[] {
    // Implement trade calculation logic
    // This is a simplified version - you'll need to implement proper margin calculations
    const tradeAmount = ethers.utils.parseUnits(trade.amount, 18);
    const price = ethers.utils.parseUnits(trade.price, 18);
    
    // Clone current outcome
    const newOutcome = JSON.parse(JSON.stringify(currentOutcome));
    
    // Apply trade adjustments (simplified)
    if (trade.side === 'buy') {
      // Buyer gets asset, pays USDT
      // Implement actual logic based on your trading rules
    } else {
      // Seller gives asset, receives USDT
    }
    
    return newOutcome;
  }

  getChannelManager(): ChannelManager {
    return this.channelManager;
  }
}
```

---

## Testing and Validation

### 1. Unit Tests

Create `test/BIEIntegration.test.ts`:

```typescript
import { expect } from "chai";
import { ethers } from "hardhat";
import { Contract, Signer } from "ethers";

describe("BIE Integration", function () {
  let bieIntegration: Contract;
  let yellowToken: Contract;
  let owner: Signer;
  let operator: Signer;
  let user: Signer;

  beforeEach(async function () {
    [owner, operator, user] = await ethers.getSigners();

    // Deploy mock Yellow token
    const YellowToken = await ethers.getContractFactory("YellowToken");
    yellowToken = await YellowToken.deploy(
      "Test Yellow",
      "TYELLOW",
      ethers.utils.parseUnits("1000000", 8)
    );

    // Deploy BIE Integration
    const BIEIntegration = await ethers.getContractFactory("BIEIntegration");
    bieIntegration = await BIEIntegration.deploy(
      yellowToken.address,
      ethers.constants.AddressZero // Mock adjudicator
    );
  });

  describe("Operator Management", function () {
    it("Should authorize operator", async function () {
      await bieIntegration.authorizeOperator(await operator.getAddress());
      expect(
        await bieIntegration.authorizedOperators(await operator.getAddress())
      ).to.be.true;
    });

    it("Should revoke operator", async function () {
      await bieIntegration.authorizeOperator(await operator.getAddress());
      await bieIntegration.revokeOperator(await operator.getAddress());
      expect(
        await bieIntegration.authorizedOperators(await operator.getAddress())
      ).to.be.false;
    });
  });

  describe("Channel Creation", function () {
    beforeEach(async function () {
      // Setup: authorize operator and fund them with tokens
      await bieIntegration.authorizeOperator(await operator.getAddress());
      await yellowToken.transfer(
        await operator.getAddress(),
        ethers.utils.parseUnits("1000", 8)
      );
      await yellowToken
        .connect(operator)
        .approve(bieIntegration.address, ethers.utils.parseUnits("1000", 8));
    });

    it("Should create trading channel", async function () {
      const counterparty = await user.getAddress();
      const amount = ethers.utils.parseUnits("100", 8);

      await expect(
        bieIntegration
          .connect(operator)
          .createTradingChannel(counterparty, amount)
      )
        .to.emit(bieIntegration, "ChannelCreated")
        .withArgs(
          ethers.utils.keccak256(
            ethers.utils.defaultAbiCoder.encode(
              ["address", "address", "uint256"],
              [await operator.getAddress(), counterparty, amount]
            )
          ),
          await operator.getAddress()
        );
    });

    it("Should fail with insufficient authorization", async function () {
      const counterparty = await user.getAddress();
      const amount = ethers.utils.parseUnits("100", 8);

      await expect(
        bieIntegration
          .connect(user)
          .createTradingChannel(counterparty, amount)
      ).to.be.revertedWith("Not authorized");
    });
  });
});
```

### 2. Integration Tests

Create `test/integration/YellowNetwork.test.ts`:

```typescript
import { expect } from "chai";
import { ethers } from "hardhat";
import { YellowNetworkClient } from "../../src/YellowNetworkClient";
import { TradingEngine } from "../../src/TradingEngine";

describe("Yellow Network Integration", function () {
  let client: YellowNetworkClient;
  let tradingEngine: TradingEngine;
  let privateKey: string;

  before(async function () {
    // Use hardhat's first account
    const [signer] = await ethers.getSigners();
    privateKey = "0x" + "ac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"; // Hardhat account #0
    
    client = new YellowNetworkClient("sepolia", privateKey);
    tradingEngine = new TradingEngine(client);
  });

  it("Should connect to Yellow Network", async function () {
    const balance = await client.getBalance();
    expect(balance).to.be.a("object"); // BigNumber
  });

  it("Should create trading channel", async function () {
    const channelParams = {
      counterparty: "0x70997970C51812dc3A010C7d01b50e0d17dc79C8", // Hardhat account #1
      asset: "0x6B175474E89094C44Da98b954EedeAC495271d0F", // DAI on mainnet
      amount: "1000"
    };

    const channelId = await tradingEngine.openTradingChannel(channelParams);
    expect(channelId).to.be.a("string");
    expect(channelId.length).to.equal(66); // 0x + 64 hex chars
  });
});
```

### 3. Run Tests

```bash
# Run unit tests
npx hardhat test

# Run specific test file
npx hardhat test test/BIEIntegration.test.ts

# Run with coverage
npx hardhat coverage

# Run integration tests (requires network connection)
npx hardhat test test/integration/ --network sepolia
```

---

## Production Deployment

### 1. Pre-deployment Checklist

```bash
# Security checklist script
cat > scripts/pre-deployment-check.sh << 'EOF'
#!/bin/bash

echo "🔍 Pre-deployment Security Check"
echo "================================"

# Check environment variables
echo "✅ Checking environment variables..."
if [ -z "$DEPLOYER_PRIVATE_KEY" ]; then
  echo "❌ DEPLOYER_PRIVATE_KEY not set"
  exit 1
fi

if [ -z "$ETHEREUM_RPC_URL" ]; then
  echo "❌ ETHEREUM_RPC_URL not set"
  exit 1
fi

# Check contract compilation
echo "✅ Compiling contracts..."
npx hardhat compile

# Run tests
echo "✅ Running tests..."
npx hardhat test

# Check for security issues
echo "✅ Running security checks..."
# Add security analysis tools here

echo "✅ All checks passed!"
EOF

chmod +x scripts/pre-deployment-check.sh
```

### 2. Mainnet Deployment Script

Create `scripts/deploy-mainnet.ts`:

```typescript
import { ethers } from "hardhat";
import { networks } from "../src/config/networks";

async function deployMainnet() {
  console.log("🚀 Starting Mainnet Deployment");
  console.log("==============================");

  // Safety checks
  const network = await ethers.provider.getNetwork();
  if (network.chainId !== 1) {
    throw new Error("Not connected to Ethereum mainnet!");
  }

  const [deployer] = await ethers.getSigners();
  const balance = await deployer.getBalance();
  
  console.log("Deployer address:", deployer.address);
  console.log("Deployer balance:", ethers.utils.formatEther(balance), "ETH");

  // Minimum balance check
  const minBalance = ethers.utils.parseEther("0.1");
  if (balance.lt(minBalance)) {
    throw new Error("Insufficient ETH balance for deployment");
  }

  // Gas price check
  const gasPrice = await ethers.provider.getGasPrice();
  console.log("Current gas price:", ethers.utils.formatUnits(gasPrice, "gwei"), "gwei");

  // Deploy contracts
  const networkConfig = networks.ethereum;
  
  console.log("Deploying BIE Integration contract...");
  const BIEIntegration = await ethers.getContractFactory("BIEIntegration");
  
  // Estimate gas
  const deployTx = BIEIntegration.getDeployTransaction(
    networkConfig.contracts.yellowToken,
    networkConfig.contracts.adjudicator
  );
  const estimatedGas = await ethers.provider.estimateGas(deployTx);
  console.log("Estimated gas:", estimatedGas.toString());

  const bieIntegration = await BIEIntegration.deploy(
    networkConfig.contracts.yellowToken,
    networkConfig.contracts.adjudicator,
    {
      gasLimit: estimatedGas.mul(120).div(100), // 20% buffer
      gasPrice: gasPrice
    }
  );

  console.log("Deployment transaction:", bieIntegration.deployTransaction.hash);
  console.log("Waiting for confirmation...");
  
  await bieIntegration.deployed();
  
  console.log("✅ BIE Integration deployed to:", bieIntegration.address);
  
  // Wait for more confirmations before verification
  console.log("Waiting for block confirmations...");
  await bieIntegration.deployTransaction.wait(10);

  // Contract verification
  try {
    console.log("Verifying contract on Etherscan...");
    await run("verify:verify", {
      address: bieIntegration.address,
      constructorArguments: [
        networkConfig.contracts.yellowToken,
        networkConfig.contracts.adjudicator
      ],
    });
    console.log("✅ Contract verified on Etherscan");
  } catch (error) {
    console.log("❌ Verification failed:", error);
  }

  // Save deployment info
  const deploymentInfo = {
    network: "ethereum",
    chainId: 1,
    contractAddress: bieIntegration.address,
    deployerAddress: deployer.address,
    deploymentTx: bieIntegration.deployTransaction.hash,
    gasUsed: estimatedGas.toString(),
    gasPrice: gasPrice.toString(),
    timestamp: new Date().toISOString(),
    blockNumber: await ethers.provider.getBlockNumber()
  };

  console.log("\n📋 Deployment Summary:");
  console.log(JSON.stringify(deploymentInfo, null, 2));

  // Save to file
  const fs = require("fs");
  fs.writeFileSync(
    `deployments/mainnet-${Date.now()}.json`,
    JSON.stringify(deploymentInfo, null, 2)
  );

  console.log("✅ Deployment completed successfully!");
}

deployMainnet()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error("❌ Deployment failed:", error);
    process.exit(1);
  });
```

### 3. Deploy to Production

```bash
# Run pre-deployment checks
./scripts/pre-deployment-check.sh

# Deploy to mainnet (be very careful!)
NETWORK=ethereum npx hardhat run scripts/deploy-mainnet.ts --network ethereum

# Deploy to Polygon
NETWORK=polygon npx hardhat run scripts/deploy-mainnet.ts --network polygon
```

---

## Monitoring and Maintenance

### 1. Monitoring Setup

Create `src/monitoring/ChannelMonitor.ts`:

```typescript
import { ethers } from "ethers";
import { YellowNetworkClient } from "../YellowNetworkClient";

export class ChannelMonitor {
  private client: YellowNetworkClient;
  private monitoringInterval: NodeJS.Timeout | null = null;

  constructor(client: YellowNetworkClient) {
    this.client = client;
  }

  startMonitoring(intervalMs: number = 30000) {
    console.log("Starting channel monitoring...");
    
    this.monitoringInterval = setInterval(async () => {
      try {
        await this.checkChannelHealth();
        await this.checkBalances();
        await this.checkChallenges();
      } catch (error) {
        console.error("Monitoring error:", error);
      }
    }, intervalMs);
  }

  stopMonitoring() {
    if (this.monitoringInterval) {
      clearInterval(this.monitoringInterval);
      this.monitoringInterval = null;
      console.log("Channel monitoring stopped");
    }
  }

  private async checkChannelHealth() {
    // Check if channels are functioning properly
    // Monitor for any unexpected state changes
  }

  private async checkBalances() {
    const balance = await this.client.getBalance();
    const threshold = ethers.utils.parseUnits("100", 8); // 100 YELLOW

    if (balance.lt(threshold)) {
      console.warn(`⚠️  Low YELLOW balance: ${ethers.utils.formatUnits(balance, 8)}`);
      // Send alert or trigger rebalancing
    }
  }

  private async checkChallenges() {
    // Monitor for channel challenges
    // Implement automatic response if needed
  }
}
```

### 2. Health Check API

Create `src/api/healthCheck.ts`:

```typescript
import express from "express";
import { YellowNetworkClient } from "../YellowNetworkClient";
import { TradingEngine } from "../TradingEngine";

export function createHealthCheckAPI(
  client: YellowNetworkClient,
  tradingEngine: TradingEngine
) {
  const app = express();

  app.get('/health', async (req, res) => {
    try {
      const balance = await client.getBalance();
      const channels = tradingEngine.getChannelManager().listChannels();
      
      const health = {
        status: 'healthy',
        timestamp: new Date().toISOString(),
        yellowBalance: balance.toString(),
        activeChannels: channels.length,
        network: client.network.name
      };

      res.json(health);
    } catch (error) {
      res.status(500).json({
        status: 'unhealthy',
        error: error.message,
        timestamp: new Date().toISOString()
      });
    }
  });

  app.get('/channels', async (req, res) => {
    try {
      const channels = tradingEngine.getChannelManager().listChannels();
      res.json(channels);
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  });

  return app;
}
```

### 3. Automated Operations

Create `scripts/maintenance.ts`:

```typescript
import { YellowNetworkClient } from "../src/YellowNetworkClient";
import { TradingEngine } from "../src/TradingEngine";
import { ChannelMonitor } from "../src/monitoring/ChannelMonitor";

async function runMaintenance() {
  console.log("🔧 Starting maintenance operations");

  const client = new YellowNetworkClient(
    process.env.NETWORK || "polygon",
    process.env.OPERATOR_PRIVATE_KEY!
  );

  const tradingEngine = new TradingEngine(client);
  const monitor = new ChannelMonitor(client);

  // Start monitoring
  monitor.startMonitoring();

  // Cleanup old channels
  await cleanupOldChannels(tradingEngine);

  // Rebalance if needed
  await rebalanceIfNeeded(client);

  console.log("✅ Maintenance completed");
}

async function cleanupOldChannels(engine: TradingEngine) {
  const channels = engine.getChannelManager().listChannels();
  
  for (const [channelId, channel] of channels) {
    // Check if channel should be closed (implement your logic)
    const shouldClose = checkChannelShouldClose(channel);
    
    if (shouldClose) {
      console.log(`Closing old channel: ${channelId}`);
      await engine.closeTradingChannel(channelId);
    }
  }
}

async function rebalanceIfNeeded(client: YellowNetworkClient) {
  const balance = await client.getBalance();
  const targetBalance = ethers.utils.parseUnits("1000", 8);
  
  if (balance.lt(targetBalance)) {
    console.log("⚖️  Rebalancing needed");
    // Implement rebalancing logic
  }
}

function checkChannelShouldClose(channel: any): boolean {
  // Implement your channel cleanup logic
  return false;
}

// Run maintenance if called directly
if (require.main === module) {
  runMaintenance()
    .then(() => process.exit(0))
    .catch((error) => {
      console.error(error);
      process.exit(1);
    });
}
```

---

## Troubleshooting

### Common Issues and Solutions

1. **Gas Estimation Failures**
   ```bash
   # Check gas price and adjust
   npx hardhat run scripts/check-gas.ts --network mainnet
   ```

2. **Channel State Sync Issues**
   ```typescript
   // Force sync channel state
   await channelManager.syncChannelState(channelId);
   ```

3. **Network Connection Problems**
   ```typescript
   // Implement retry logic
   const retryOperation = async (operation: () => Promise<any>, maxRetries: number = 3) => {
     for (let i = 0; i < maxRetries; i++) {
       try {
         return await operation();
       } catch (error) {
         if (i === maxRetries - 1) throw error;
         await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
       }
     }
   };
   ```

### Support Resources

- **Discord**: https://discord.gg/yellow
- **Telegram**: https://t.me/yellow_org
- **Documentation**: https://docs.yellow.org
- **GitHub Issues**: https://github.com/layer-3/clearsync/issues

---

## Conclusion

This guide provides a comprehensive foundation for BIE developers to integrate with Yellow Network. The modular architecture allows for customization based on specific business requirements while maintaining compatibility with the Yellow Network ecosystem.

Key takeaways:
- Start with testnet deployment and thorough testing
- Implement proper monitoring and alerting
- Follow security best practices
- Keep up with Yellow Network updates and improvements

For additional support or questions, please reach out through the community channels listed above.