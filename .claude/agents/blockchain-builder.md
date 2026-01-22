# Blockchain Builder Agent (Solidity/Hardhat)

Invocable agent during Phase 3 development sessions to implement Solidity smart contracts with Hardhat, including testing and deployment scripts.

## Invocation

From a session, invoke with Task tool:

```
Task: Blockchain Builder Agent
Prompt: |
  Contract: [contract name, e.g., TokenVault, StakingPool]
  Type: [ERC20, ERC721, ERC1155, Custom]
  Functions: |
    - [function signatures and descriptions]
  Storage: |
    - [state variables]
  Events: |
    - [events to emit]
  Access Control: |
    - [roles and permissions]
  Context: [any relevant projectSpec context]
```

---

## Agent Workflow

### Step 1: Understand Contract Requirements

Parse the invocation to extract:
- Contract name and type (ERC standard if applicable)
- Required functions and their logic
- State variables and mappings
- Events to emit
- Access control requirements
- Integration points with other contracts

### Step 2: Check Existing Contracts

Scan the project:

```bash
# Check existing contracts
ls contracts/

# Check existing interfaces
ls contracts/interfaces/

# Check libraries
ls contracts/libraries/

# Check test files
ls test/
```

### Step 3: Identify Dependencies

Determine required OpenZeppelin imports:

```solidity
// Common imports
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/security/Pausable.sol";
import "@openzeppelin/contracts/utils/math/Math.sol";
```

Install if needed:
```bash
npm install @openzeppelin/contracts
```

### Step 4: Implement Contract

```solidity
// contracts/[ContractName].sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/security/Pausable.sol";

/**
 * @title TokenVault
 * @notice Secure vault for token deposits and withdrawals
 * @dev Implements role-based access control and reentrancy protection
 */
contract TokenVault is AccessControl, ReentrancyGuard, Pausable {
    using SafeERC20 for IERC20;

    // ============ Constants ============

    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN_ROLE");
    bytes32 public constant OPERATOR_ROLE = keccak256("OPERATOR_ROLE");

    // ============ State Variables ============

    /// @notice The token this vault accepts
    IERC20 public immutable token;

    /// @notice User balances
    mapping(address => uint256) public balances;

    /// @notice Total deposited amount
    uint256 public totalDeposits;

    /// @notice Minimum deposit amount
    uint256 public minDeposit;

    /// @notice Maximum deposit per user
    uint256 public maxDepositPerUser;

    // ============ Events ============

    event Deposited(address indexed user, uint256 amount, uint256 timestamp);
    event Withdrawn(address indexed user, uint256 amount, uint256 timestamp);
    event MinDepositUpdated(uint256 oldValue, uint256 newValue);
    event MaxDepositUpdated(uint256 oldValue, uint256 newValue);
    event EmergencyWithdraw(address indexed admin, uint256 amount);

    // ============ Errors ============

    error InvalidAmount();
    error InsufficientBalance();
    error DepositTooSmall(uint256 amount, uint256 minimum);
    error DepositExceedsLimit(uint256 newTotal, uint256 limit);
    error ZeroAddress();

    // ============ Constructor ============

    /**
     * @notice Initialize the vault
     * @param _token The ERC20 token address
     * @param _minDeposit Minimum deposit amount
     * @param _maxDepositPerUser Maximum deposit per user
     */
    constructor(
        address _token,
        uint256 _minDeposit,
        uint256 _maxDepositPerUser
    ) {
        if (_token == address(0)) revert ZeroAddress();

        token = IERC20(_token);
        minDeposit = _minDeposit;
        maxDepositPerUser = _maxDepositPerUser;

        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(ADMIN_ROLE, msg.sender);
    }

    // ============ External Functions ============

    /**
     * @notice Deposit tokens into the vault
     * @param amount Amount of tokens to deposit
     */
    function deposit(uint256 amount) external nonReentrant whenNotPaused {
        if (amount == 0) revert InvalidAmount();
        if (amount < minDeposit) revert DepositTooSmall(amount, minDeposit);

        uint256 newBalance = balances[msg.sender] + amount;
        if (newBalance > maxDepositPerUser) {
            revert DepositExceedsLimit(newBalance, maxDepositPerUser);
        }

        balances[msg.sender] = newBalance;
        totalDeposits += amount;

        token.safeTransferFrom(msg.sender, address(this), amount);

        emit Deposited(msg.sender, amount, block.timestamp);
    }

    /**
     * @notice Withdraw tokens from the vault
     * @param amount Amount of tokens to withdraw
     */
    function withdraw(uint256 amount) external nonReentrant whenNotPaused {
        if (amount == 0) revert InvalidAmount();
        if (balances[msg.sender] < amount) revert InsufficientBalance();

        balances[msg.sender] -= amount;
        totalDeposits -= amount;

        token.safeTransfer(msg.sender, amount);

        emit Withdrawn(msg.sender, amount, block.timestamp);
    }

    /**
     * @notice Get user's deposited balance
     * @param user User address
     * @return User's balance
     */
    function balanceOf(address user) external view returns (uint256) {
        return balances[user];
    }

    // ============ Admin Functions ============

    /**
     * @notice Update minimum deposit amount
     * @param _minDeposit New minimum deposit
     */
    function setMinDeposit(uint256 _minDeposit) external onlyRole(ADMIN_ROLE) {
        uint256 oldValue = minDeposit;
        minDeposit = _minDeposit;
        emit MinDepositUpdated(oldValue, _minDeposit);
    }

    /**
     * @notice Update maximum deposit per user
     * @param _maxDepositPerUser New maximum deposit
     */
    function setMaxDepositPerUser(uint256 _maxDepositPerUser) external onlyRole(ADMIN_ROLE) {
        uint256 oldValue = maxDepositPerUser;
        maxDepositPerUser = _maxDepositPerUser;
        emit MaxDepositUpdated(oldValue, _maxDepositPerUser);
    }

    /**
     * @notice Pause the contract
     */
    function pause() external onlyRole(ADMIN_ROLE) {
        _pause();
    }

    /**
     * @notice Unpause the contract
     */
    function unpause() external onlyRole(ADMIN_ROLE) {
        _unpause();
    }

    /**
     * @notice Emergency withdraw all tokens (admin only)
     * @dev Only use in emergency situations
     */
    function emergencyWithdraw() external onlyRole(DEFAULT_ADMIN_ROLE) {
        uint256 balance = token.balanceOf(address(this));
        token.safeTransfer(msg.sender, balance);
        emit EmergencyWithdraw(msg.sender, balance);
    }
}
```

### Step 5: Create Interface (if needed)

```solidity
// contracts/interfaces/ITokenVault.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

interface ITokenVault {
    // Events
    event Deposited(address indexed user, uint256 amount, uint256 timestamp);
    event Withdrawn(address indexed user, uint256 amount, uint256 timestamp);

    // Functions
    function deposit(uint256 amount) external;
    function withdraw(uint256 amount) external;
    function balanceOf(address user) external view returns (uint256);
    function totalDeposits() external view returns (uint256);
}
```

### Step 6: Implement Tests

```typescript
// test/TokenVault.test.ts
import { expect } from "chai";
import { ethers } from "hardhat";
import { loadFixture } from "@nomicfoundation/hardhat-toolbox/network-helpers";
import { TokenVault, MockERC20 } from "../typechain-types";
import { SignerWithAddress } from "@nomicfoundation/hardhat-ethers/signers";

describe("TokenVault", function () {
  // Test fixture
  async function deployVaultFixture() {
    const [owner, user1, user2] = await ethers.getSigners();

    // Deploy mock token
    const MockToken = await ethers.getContractFactory("MockERC20");
    const token = await MockToken.deploy("Test Token", "TEST");

    // Deploy vault
    const minDeposit = ethers.parseEther("1");
    const maxDeposit = ethers.parseEther("1000");

    const TokenVault = await ethers.getContractFactory("TokenVault");
    const vault = await TokenVault.deploy(
      await token.getAddress(),
      minDeposit,
      maxDeposit
    );

    // Mint tokens to users
    await token.mint(user1.address, ethers.parseEther("10000"));
    await token.mint(user2.address, ethers.parseEther("10000"));

    // Approve vault
    await token.connect(user1).approve(vault.getAddress(), ethers.MaxUint256);
    await token.connect(user2).approve(vault.getAddress(), ethers.MaxUint256);

    return { vault, token, owner, user1, user2, minDeposit, maxDeposit };
  }

  describe("Deployment", function () {
    it("Should set the correct token", async function () {
      const { vault, token } = await loadFixture(deployVaultFixture);
      expect(await vault.token()).to.equal(await token.getAddress());
    });

    it("Should set the correct admin", async function () {
      const { vault, owner } = await loadFixture(deployVaultFixture);
      const ADMIN_ROLE = await vault.ADMIN_ROLE();
      expect(await vault.hasRole(ADMIN_ROLE, owner.address)).to.be.true;
    });
  });

  describe("Deposits", function () {
    it("Should allow deposits above minimum", async function () {
      const { vault, user1 } = await loadFixture(deployVaultFixture);
      const amount = ethers.parseEther("10");

      await expect(vault.connect(user1).deposit(amount))
        .to.emit(vault, "Deposited")
        .withArgs(user1.address, amount, anyValue);

      expect(await vault.balanceOf(user1.address)).to.equal(amount);
    });

    it("Should reject deposits below minimum", async function () {
      const { vault, user1, minDeposit } = await loadFixture(deployVaultFixture);
      const amount = minDeposit - 1n;

      await expect(vault.connect(user1).deposit(amount))
        .to.be.revertedWithCustomError(vault, "DepositTooSmall");
    });

    it("Should reject deposits exceeding user limit", async function () {
      const { vault, user1, maxDeposit } = await loadFixture(deployVaultFixture);
      const amount = maxDeposit + 1n;

      await expect(vault.connect(user1).deposit(amount))
        .to.be.revertedWithCustomError(vault, "DepositExceedsLimit");
    });
  });

  describe("Withdrawals", function () {
    it("Should allow withdrawals up to balance", async function () {
      const { vault, user1 } = await loadFixture(deployVaultFixture);
      const depositAmount = ethers.parseEther("100");
      const withdrawAmount = ethers.parseEther("50");

      await vault.connect(user1).deposit(depositAmount);
      await expect(vault.connect(user1).withdraw(withdrawAmount))
        .to.emit(vault, "Withdrawn")
        .withArgs(user1.address, withdrawAmount, anyValue);

      expect(await vault.balanceOf(user1.address)).to.equal(
        depositAmount - withdrawAmount
      );
    });

    it("Should reject withdrawals exceeding balance", async function () {
      const { vault, user1 } = await loadFixture(deployVaultFixture);
      const depositAmount = ethers.parseEther("100");
      const withdrawAmount = ethers.parseEther("150");

      await vault.connect(user1).deposit(depositAmount);
      await expect(vault.connect(user1).withdraw(withdrawAmount))
        .to.be.revertedWithCustomError(vault, "InsufficientBalance");
    });
  });

  describe("Admin Functions", function () {
    it("Should allow admin to pause", async function () {
      const { vault, owner, user1 } = await loadFixture(deployVaultFixture);

      await vault.connect(owner).pause();

      await expect(vault.connect(user1).deposit(ethers.parseEther("10")))
        .to.be.revertedWithCustomError(vault, "EnforcedPause");
    });

    it("Should reject non-admin from pausing", async function () {
      const { vault, user1 } = await loadFixture(deployVaultFixture);

      await expect(vault.connect(user1).pause())
        .to.be.revertedWithCustomError(vault, "AccessControlUnauthorizedAccount");
    });
  });
});

// Helper for any value matching
const anyValue = () => true;
```

### Step 7: Create Deployment Script

```typescript
// scripts/deploy/[contract].ts
import { ethers, run, network } from "hardhat";

async function main() {
  console.log("Deploying TokenVault...");

  const [deployer] = await ethers.getSigners();
  console.log("Deploying with account:", deployer.address);

  // Configuration
  const TOKEN_ADDRESS = process.env.TOKEN_ADDRESS!;
  const MIN_DEPOSIT = ethers.parseEther("1");
  const MAX_DEPOSIT = ethers.parseEther("10000");

  // Deploy
  const TokenVault = await ethers.getContractFactory("TokenVault");
  const vault = await TokenVault.deploy(
    TOKEN_ADDRESS,
    MIN_DEPOSIT,
    MAX_DEPOSIT
  );

  await vault.waitForDeployment();
  const vaultAddress = await vault.getAddress();

  console.log("TokenVault deployed to:", vaultAddress);

  // Verify on Etherscan (if not local)
  if (network.name !== "hardhat" && network.name !== "localhost") {
    console.log("Waiting for block confirmations...");
    await vault.deploymentTransaction()?.wait(5);

    console.log("Verifying contract...");
    await run("verify:verify", {
      address: vaultAddress,
      constructorArguments: [TOKEN_ADDRESS, MIN_DEPOSIT, MAX_DEPOSIT],
    });
  }

  // Output deployment info
  console.log("\n=== Deployment Summary ===");
  console.log("Network:", network.name);
  console.log("TokenVault:", vaultAddress);
  console.log("Token:", TOKEN_ADDRESS);
  console.log("Min Deposit:", ethers.formatEther(MIN_DEPOSIT), "tokens");
  console.log("Max Deposit:", ethers.formatEther(MAX_DEPOSIT), "tokens");
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

### Step 8: Create TypeScript Types for Frontend Integration

```typescript
// typechain-types/contracts.ts (generated, but example of usage)

// Usage in NestJS backend
import { ethers } from "ethers";
import { TokenVault__factory } from "./typechain-types";

const provider = new ethers.JsonRpcProvider(process.env.RPC_URL);
const wallet = new ethers.Wallet(process.env.PRIVATE_KEY!, provider);

const vault = TokenVault__factory.connect(VAULT_ADDRESS, wallet);

// Read balance
const balance = await vault.balanceOf(userAddress);

// Write transaction
const tx = await vault.deposit(amount);
await tx.wait();
```

### Step 9: Update Hardhat Config (if needed)

```typescript
// hardhat.config.ts
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";
import "dotenv/config";

const config: HardhatUserConfig = {
  solidity: {
    version: "0.8.20",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200,
      },
    },
  },
  networks: {
    hardhat: {
      chainId: 31337,
    },
    sepolia: {
      url: process.env.SEPOLIA_RPC_URL || "",
      accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : [],
    },
    polygon: {
      url: process.env.POLYGON_RPC_URL || "",
      accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : [],
    },
  },
  etherscan: {
    apiKey: {
      sepolia: process.env.ETHERSCAN_API_KEY || "",
      polygon: process.env.POLYGONSCAN_API_KEY || "",
    },
  },
  gasReporter: {
    enabled: process.env.REPORT_GAS === "true",
    currency: "USD",
  },
};

export default config;
```

---

## Security Checklist

Before deployment, verify:

- [ ] **Reentrancy**: Use ReentrancyGuard for external calls
- [ ] **Integer Overflow**: Solidity 0.8+ has built-in checks
- [ ] **Access Control**: Proper role-based permissions
- [ ] **Input Validation**: Check all parameters
- [ ] **External Calls**: Use SafeERC20, check return values
- [ ] **Front-running**: Consider if transactions can be front-run
- [ ] **Gas Limits**: Avoid unbounded loops
- [ ] **Upgradability**: Consider proxy pattern if needed
- [ ] **Events**: Emit events for all state changes
- [ ] **Documentation**: NatSpec comments for all public functions

---

## Agent Output

Upon completion, the agent should have created:

1. **Contract file** with full implementation
2. **Interface file** (if applicable)
3. **Test file** with comprehensive coverage
4. **Deployment script**
5. **Updated hardhat.config.ts** (if needed)

And report:
- Created files list
- Contract functions summary
- Gas estimates for main operations
- Security considerations
- Integration notes for backend

---

## Invocation Example

```
Task: Blockchain Builder Agent
Prompt: |
  Contract: StakingPool
  Type: Custom
  Functions: |
    - stake(uint256 amount): Lock tokens for staking
    - unstake(uint256 amount): Withdraw staked tokens
    - claimRewards(): Claim accumulated rewards
    - getRewardRate(): View current reward rate
    - getStakedBalance(address): View user's staked amount
    - getPendingRewards(address): View pending rewards
  Storage: |
    - stakedBalances: mapping(address => uint256)
    - rewardDebt: mapping(address => uint256)
    - totalStaked: uint256
    - rewardPerToken: uint256
    - lastUpdateTime: uint256
  Events: |
    - Staked(address user, uint256 amount)
    - Unstaked(address user, uint256 amount)
    - RewardsClaimed(address user, uint256 amount)
  Access Control: |
    - Owner can set reward rate
    - Owner can pause/unpause
    - Anyone can stake/unstake/claim
  Context: |
    See projectSpec section for tokenomics
    Reward token is separate from staking token
    Rewards calculated per second based on share of pool
```

---

## Important Notes

- **Always use latest Solidity** - 0.8.20+ for built-in overflow protection
- **OpenZeppelin contracts** - Use audited base contracts
- **Test coverage** - Aim for >90% coverage
- **Gas optimization** - Use view functions, pack storage
- **Events** - Emit for all state changes (indexers need this)
- **NatSpec** - Document all public functions
- **Security audit** - Get professional audit before mainnet
