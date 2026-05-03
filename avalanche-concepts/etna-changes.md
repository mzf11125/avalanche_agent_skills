# Etna Upgrade & ACP-77

## What Is Etna?

Etna is a major Avalanche network upgrade that activated in late 2024. Its most significant change - defined in **ACP-77** (Avalanche Community Proposal 77) - fundamentally changed how Avalanche L1 validators work, reducing the cost of running an L1 by ~99.9%.

## What ACP-77 Changed

### Before ACP-77

Every validator of an Avalanche L1 (then called a "Subnet") was **required** to also validate the Avalanche Primary Network. This meant:

- Each L1 validator had to stake **2,000 AVAX** on the P-Chain
- At peak AVAX prices (~$50), this was **$100,000 per validator**
- A 5-validator L1 required $500,000 locked in stake
- This was the single biggest barrier to L1 deployment

### After ACP-77

L1 validators are **no longer required** to validate the primary network. Instead:

- L1 validators pay a small **continuous fee** in AVAX (pay-as-you-go)
- The fee is proportional to the number of active L1 validators
- No minimum AVAX stake required for L1-only validators
- Cost reduction: **~99.9%** compared to pre-Etna

### Continuous Fee Model

The continuous fee is calculated based on the number of active L1 validators and a base fee rate set by the primary network. Fees are paid from a balance held in a **PoA or PoS Validator Manager contract** on the L1.

```
monthly_fee ≈ num_validators × base_rate_per_validator
```

The exact rate is governed by the primary network and can change via ACP.

## Validator Manager Contracts

Post-Etna, L1 validators are managed by on-chain contracts:

### PoA Validator Manager (Proof of Authority)

- An admin address controls who can be a validator
- Suitable for permissioned L1s (enterprise, gaming, etc.)
- Validators are added/removed by the admin
- No token staking required from validators

### PoS Validator Manager (Proof of Stake)

- Validators stake the L1's native token
- Permissionless - anyone with enough stake can become a validator
- Supports delegation
- Suitable for decentralized L1s

## Nomenclature Change: Subnet → Avalanche L1

With Etna, Avalanche officially renamed "Subnets" to "Avalanche L1s":

| Old Term | New Term |
|----------|----------|
| Subnet | Avalanche L1 |
| Subnet validator | L1 validator |
| Subnet-EVM | Subnet-EVM (unchanged - it's the VM name) |
| Primary Network Subnet | Primary Network (unchanged) |

The term "Subnet" still appears in older documentation, code, and tooling. They refer to the same concept.

## What Stayed the Same

- The three-chain primary network (C/P/X) is unchanged
- Subnet-EVM is unchanged (still the standard VM for L1s)
- ICM/Teleporter is unchanged
- ICTT is unchanged
- AvalancheGo is unchanged (just updated to support new validator model)
- Fuji testnet supports the new model

## Migration Path for Existing Subnets

Existing Subnets (pre-Etna) can migrate to the new model:

1. Deploy a Validator Manager contract on the L1
2. Call the migration function on the P-Chain
3. Existing validators transition to the continuous fee model
4. The 2,000 AVAX stake is unlocked and returned

Migration is optional - existing Subnets continue to work under the old model until they choose to migrate.

## Impact on L1 Deployment Cost

| Scenario | Pre-Etna Cost | Post-Etna Cost |
|----------|--------------|----------------|
| 5-validator L1 (stake) | ~$500,000 locked | ~$0 locked |
| Monthly operational cost | Opportunity cost of locked AVAX | ~$50–200/month in fees |
| Minimum viable L1 | 1 validator × 2,000 AVAX | 1 validator × continuous fee |

This is why Retro9000 saw a surge in L1 projects after Etna - the economic barrier was essentially eliminated.
