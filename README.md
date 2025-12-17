---
eip: XXX
title: Conditional Tokens
description: An interface for tokens representing positions on outcomes that can be split, merged and redeemed based on oracle reported results
author: shafu (@shafu0x)
status: Draft
type: Standards Track
category: ERC
created: 2025-12-16
requires: 6909
---

## Abstract

This ERC extends ERC-6909 with conditional tokens that allow participants to create and settle positions on future outcomes.

It introduces three core operations. Splitting collateral into outcome positions, merging positions back into collateral and redeeming positions after oracle resolution.

## Motivation

Prediction markets have demonstrated product market fit through platform like Polymarket. The Gnosis Conditional Tokens framework from 2019 pioneered the core primitives of splitting, merging and redemeeing positions based on oracle outcomes. But there is no formal ERC standard, limiting interoperability.

To enable a thriving ecosystem of prediction markets and futarchy governance we need a standard interface. This ERC addresses this through three core operations:

1. **Condition Preparation**: Registers a condition with an oracle, question identifier and outcome count.
2. **Position Splitting & Merging**: Converts collateral into outcome tokens (split) or recombines them (merge).
3. **Redemptions**: Token holders can claim collateral proportional to the reported payout weights after oracle resolution.

This ERC formalizes patterns that the prediction market industry has battle-tested for years now. Providing one interface will accelerate adoption accross chains and applications.

## Specification

### Methods

#### prepareCondition

Initialize a new condition with a fixed number of outcomes. The function generates a `conditionId` which is the `keccak256(abi.encodePacked(oracle, questionId, outcomeSlotCount))` and initializes a payout vector associated to the `conditionId`

```js
function prepareCondition(address oracle, bytes32 questionId, uint outcomeSlotCount) external
```

#### reportPayouts

Oracle resolves a condition by calling this function and reports payouts for each outcome

**NOTE**:
`msg.sender` is enforced as the oracle, because conditionId is derived from `(msg.sender, questionId, payouts.length)`.

```js
function reportPayouts(bytes32 questionId, uint[] calldata payouts) external
```

#### splitPosition

Convert one `parent` stake into multiple `child` outcome positions, either by collateral by transferring `amount` collateral from the message sender to itself or by burning `amount` stake held by the message sender in the position being split worth of EIP 1155 tokens.

```js
function splitPosition(
        IERC20 collateralToken,
        bytes32 parentCollectionId,
        bytes32 conditionId,
        uint[] calldata partition,
        uint amount
    ) external
```

#### mergePositions

The inverse of `splitPosition`: burn multiple child positions to recreate a parent position or get back collateral.

```js
function mergePositions(
        IERC20 collateralToken,
        bytes32 parentCollectionId,
        bytes32 conditionId,
        uint[] calldata partition,
        uint amount
    ) external
```

#### redeemPositions

After a condition is resolved, redeem outcome position tokens for their payout share.

```js
function redeemPositions(IERC20 collateralToken, bytes32 parentCollectionId, bytes32 conditionId, uint[] calldata indexSets) external
```

#### getOutcomeSlotCount

Returns outcome slot count of a `conditionId`

```js
function getOutcomeSlotCount(bytes32 conditionId) external view returns (uint)
```

#### getConditionId

Returns generated `conditionId` which is the `keccak256(abi.encodePacked(oracle, questionId, outcomeSlotCount))`

```js
function getConditionId(address oracle, bytes32 questionId, uint outcomeSlotCount) external pure returns (bytes32)
```

#### getCollectionId

Returns `collectionId` constructed by a parent collection and an outcome collection.

```js
function getCollectionId(bytes32 parentCollectionId, bytes32 conditionId, uint indexSet) external view returns (bytes32)
```

#### getPositionId

Returns positionID from collateral token and outcome collection associated to the position

```js
function getPositionId(IERC20 collateralToken, bytes32 collectionId) external pure returns (uint)
```
