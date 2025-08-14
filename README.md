
# Solana Token & Raydium Liquidity Management Library

This library provides an easy-to-use interface for creating SPL tokens, managing liquidity pools on Raydium, performing swaps, and handling LP tokens.  
It is designed for developers building projects on the Solana blockchain using Node.js.

---

## **A. Token Management**

### `createToken(name, symbol, decimals, initialSupply)`
- Mints a new SPL token.
- Mints initial supply to creator wallet.
- Disables mint authority for trust.

### `createATA(wallet, mintAddress)`
- Creates an Associated Token Account (ATA) for holding the token.

### `mintTokens(mintAddress, destination, amount)`
- Mints more tokens to a given ATA (only if mint authority exists).

### `disableMintAuthority(mintAddress)`
- Permanently disables minting.

---

## **B. Liquidity Pool (Raydium)**

*(These functions wrap Raydium AMM program calls)*

### `createPool(tokenMint, baseMint, tokenAmount, baseAmount)`
- Initializes a new pool with token + base (SOL or USDC).
- Calculates initial price.
- Returns LP token mint and address.

### `addLiquidity(poolId, tokenAmount, baseAmount)`
- Adds more liquidity to an existing pool.
- Mints LP tokens to the provider.

### `removeLiquidity(poolId, lpAmount)`
- Burns LP tokens.
- Returns proportional SOL + token reserves to the wallet.

### `getPoolInfo(poolId)`
- Returns reserves, LP supply, price, volume.

---

## **C. Swapping**

### `swapBaseToToken(poolId, baseAmount, minTokenOut)`
- Swap SOL/USDC for your token.

### `swapTokenToBase(poolId, tokenAmount, minBaseOut)`
- Swap your token for SOL/USDC.

### `getSwapQuote(poolId, amountIn, direction)`
- Get how many tokens you’ll receive for a given swap.
- Uses Raydium's constant product formula:
```
amountOut = (amountIn * reserveOut) / (reserveIn + amountIn)
```

---

## **D. Utility / Admin**

### `getLPBalance(wallet, lpMint)`
- Shows LP token balance for the user.

### `getPrice(poolId)`
- Returns current token price in base currency.

### `withdrawAllLiquidity(poolId)`
- Burns all LP tokens in wallet and withdraws liquidity.

---

## **Rust Function Signatures**

```rust
// =========================
// A. TOKEN MANAGEMENT
// =========================

pub fn create_token(
    name: &str,
    symbol: &str,
    decimals: u8,
    initial_supply: u64,
) -> Result<Pubkey, ProgramError>;

pub fn create_ata(
    wallet: &Pubkey,
    mint_address: &Pubkey,
) -> Result<Pubkey, ProgramError>;

pub fn mint_tokens(
    mint_address: &Pubkey,
    destination: &Pubkey,
    amount: u64,
) -> Result<(), ProgramError>;

pub fn disable_mint_authority(
    mint_address: &Pubkey,
) -> Result<(), ProgramError>;


// =========================
// B. LIQUIDITY POOL (Raydium)
// =========================

pub fn create_pool(
    token_mint: &Pubkey,
    base_mint: &Pubkey,
    token_amount: u64,
    base_amount: u64,
) -> Result<Pubkey, ProgramError>;

pub fn add_liquidity(
    pool_id: &Pubkey,
    token_amount: u64,
    base_amount: u64,
) -> Result<(), ProgramError>;

pub fn remove_liquidity(
    pool_id: &Pubkey,
    lp_amount: u64,
) -> Result<(), ProgramError>;

pub fn get_pool_info(
    pool_id: &Pubkey,
) -> Result<PoolInfo, ProgramError>;


// =========================
// C. SWAPPING
// =========================

pub fn swap_base_to_token(
    pool_id: &Pubkey,
    base_amount: u64,
    min_token_out: u64,
) -> Result<(), ProgramError>;

pub fn swap_token_to_base(
    pool_id: &Pubkey,
    token_amount: u64,
    min_base_out: u64,
) -> Result<(), ProgramError>;

pub fn get_swap_quote(
    pool_id: &Pubkey,
    amount_in: u64,
    direction: SwapDirection,
) -> Result<u64, ProgramError>;


// =========================
// D. UTILITY / ADMIN
// =========================

pub fn get_lp_balance(
    wallet: &Pubkey,
    lp_mint: &Pubkey,
) -> Result<u64, ProgramError>;

pub fn get_price(
    pool_id: &Pubkey,
) -> Result<f64, ProgramError>;

pub fn withdraw_all_liquidity(
    pool_id: &Pubkey,
) -> Result<(), ProgramError>;


// =========================
// DATA STRUCTURES
// =========================

pub struct PoolInfo {
    pub reserve_token: u64,
    pub reserve_base: u64,
    pub lp_supply: u64,
    pub price: f64,
    pub volume_24h: f64,
}

pub enum SwapDirection {
    BaseToToken,
    TokenToBase,
}
```

---

### **Notes**

* `Pubkey` is the Solana public key type.
* `ProgramError` is the standard Solana program error type.
* `Result<_, ProgramError>` ensures proper error handling in on-chain programs.
* `PoolInfo` struct holds relevant pool data.
* `SwapDirection` enum clarifies swap direction without needing a bool flag.
