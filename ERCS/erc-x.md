---
eip: x
title: Cancelation for ERC-7540 Tokenized Vaults
description: Extension of ERC-7540 with cancelation support
author: Jeroen Offerijns (@hieronx), Alina Sinelnikova (@ilinzweilin), Vikram Arun (@vikramarun), Joey Santoro (@joeysantoro), Farhaan Ali (@0xfarhaan)
discussions-to: TODO
status: Draft
type: Standards Track
category: ERC
created: 2024-04-07
requires: 165, 7540
---

## Abstract

The following standard extends [ERC-7540](./eip-7540.md) by adding support for asynchronous cancelation flows.

## Motivation

Shares or assets locked for Requests can be stuck in the Pending state. For some use cases, such as redeeming from a pool of long-dated real-world assets, this can take a considerable amount of time.

This standard expands the scope Asynchronous ERC-7540 Vaults by adding cancelation support.

## Specification

### Definitions:

The existing definitions from [ERC-7540](./eip-7540.md) apply.

### Cancelation Lifecycle

After submission, cancelation Requests go through Pending, Claimable, and Claimed stages. An example lifecycle for a deposit cancelation Request is visualized in the table below.

| **State**   | **User**                         | **Vault** |
|-------------|---------------------------------|-----------|
| Pending     | `cancelDepositRequest(requestId, owner)` | `pendingCancelDepositRequest[owner] = true` |
| Claimable   |                                 | *Internal cancelation fulfillment*:  `pendingCancelDepositRequest[owner] = false`; `claimableCancelDepositRequest[owner] = assets` |
| Claimed     | `claimCancelDepositRequest(requestId, receiver, owner)`      | `claimableDepositRequest[owner] -= assets`; `asset.balanceOf[receiver] += assets` |

Requests MUST NOT skip or otherwise short-circuit the Claim state. In other words, to initiate and claim a Request, a user MUST call both cancel* and the corresponding Claim function separately, even in the same block. Vaults MUST NOT "push" tokens onto the user after a Request, users MUST "pull" the tokens via the Claim function.

While a deposit cancelation Request is Pending, new deposit Requests are blocked. Likewise, while a redeem cancelation Request is Pending, new redeem Requests are blocked.

### Methods

#### cancelDepositRequest

Submits a Request for asynchronous deposit cancelation. This places the Request in Pending state, with a corresponding increase in `pendingCancelDepositRequest` for the full amount of the pending deposit Request. 

When the cancelation is Pending, new deposit Requests are blocked and `requestDeposit` MUST revert.

When the cancelation is Claimable, `claimableCancelDepositRequest` will be increased for the `owner`. `claimCancelDepositRequest` can subsequently be called by `owner` to receive `assets`. A Request MAY transition straight to Claimable state but MUST NOT skip the Claimable state.

`owner` MUST equal `msg.sender` unless the `owner` has approved the `msg.sender` by some mechanism. 

MUST emit the `CancelDepositRequest` event.

```yaml
- name: cancelDepositRequest
  type: function
  stateMutability: nonpayable

  inputs:
    - name: requestId
      type: uint256
    - name: owner
      type: address
  outputs:
```

#### pendingCancelDepositRequest

Whether the given `requestId` and `owner` have a pending deposit cancelation Request.

MUST NOT show any variations depending on the caller.

MUST NOT revert unless due to integer overflow caused by an unreasonably large input.

```yaml
- name: pendingCancelDepositRequest
  type: function
  stateMutability: view

  inputs:
    - name: requestId
      type: uint256
    - name: owner
      type: address

  outputs:
    - name: isPending
      type: bool
```

#### claimableCancelDepositRequest

The amount of `assets` in Claimable cancelation state for the `owner` to claim.

MUST NOT show any variations depending on the caller.

MUST NOT revert unless due to integer overflow caused by an unreasonably large input.

```yaml
- name: claimableCancelDepositRequest
  type: function
  stateMutability: view

  inputs:
    - name: requestId
      type: uint256
    - name: owner
      type: address

  outputs:
    - name: assets
      type: uint256
```

#### claimCancelDepositRequest

Claims the deposit cancelation Request with `requestId` and `owner`.

Transfers `assets` to `receiver`.

`owner` MUST equal `msg.sender` unless the `owner` has approved the `msg.sender` by some mechanism. 

MUST emit the `ClaimCancelDepositRequest` event.

```yaml
- name: claimCancelDepositRequest
  type: function
  stateMutability: nonpayable

  inputs:
    - name: requestId
      type: uint256
    - name: receiver
      type: address
    - name: owner
      type: address
  outputs:
```

#### cancelRedeemRequest

Submits a Request for asynchronous redeem cancelation. This places the Request in Pending state, with a corresponding increase in `pendingCancelRedeemRequest` for the full amount of the pending redeem Request. 

When the cancelation is Pending, new redeem Requests are blocked and `requestRedeem` MUST revert.

When the cancelation is Claimable, `claimableCancelRedeemRequest` will be increased for the `owner`. `claimCancelRedeemRequest` can subsequently be called by `owner` to receive `shares`. A Request MAY transition straight to Claimable state but MUST NOT skip the Claimable state.

`owner` MUST equal `msg.sender` unless the `owner` has approved the `msg.sender` by some mechanism. 

MUST emit the `CancelRedeemRequest` event.

```yaml
- name: cancelRedeemRequest
  type: function
  stateMutability: nonpayable

  inputs:
    - name: requestId
      type: uint256
    - name: owner
      type: address
  outputs:
```

#### pendingCanceRedeemRequest

Whether the given `requestId` and `owner` have a pending redeem cancelation Request.

MUST NOT show any variations depending on the caller.

MUST NOT revert unless due to integer overflow caused by an unreasonably large input.

```yaml
- name: pendingCanceRedeemRequest
  type: function
  stateMutability: view

  inputs:
    - name: requestId
      type: uint256
    - name: owner
      type: address

  outputs:
    - name: isPending
      type: bool
```

#### claimableCancelRedeemRequest

The amount of `shares` in Claimable cancelation state for the `owner` to claim.

MUST NOT show any variations depending on the caller.

MUST NOT revert unless due to integer overflow caused by an unreasonably large input.

```yaml
- name: claimableCancelRedeemRequest
  type: function
  stateMutability: view

  inputs:
    - name: requestId
      type: uint256
    - name: owner
      type: address

  outputs:
    - name: shares
      type: uint256
```

#### claimCancelRedeemRequest

Claims the redeem cancelation Request with `requestId` and `owner`.

Transfers `assets` to `receiver`.

`owner` MUST equal `msg.sender` unless the `owner` has approved the `msg.sender` by some mechanism. 

MUST emit the `ClaimCancelRedeemRequest` event.

```yaml
- name: claimCancelRedeemRequest
  type: function
  stateMutability: nonpayable

  inputs:
    - name: requestId
      type: uint256
    - name: receiver
      type: address
    - name: owner
      type: address
  outputs:
```

### Events

#### CancelDepositRequest

`owner` has requested cancelation of their deposit Request with request ID `requestId`. `sender` is the caller of the `cancelDepositRequest` which may not be equal to the `owner`.

MUST be emitted when a deposit cancelation Request is submitted using the `cancelDepositRequest` method.

```yaml
- name: CancelDepositRequest
  type: event

  inputs:
    - name: owner
      indexed: true
      type: address
    - name: requestId
      indexed: true
      type: uint256
    - name: sender
      indexed: false
      type: address
```

#### CancelDepositClaim

`owner` has claimed their deposit cancelation Request with request ID `requestId`. `receiver` is the destination of the `assets`. `sender` is the caller of the `claimCancelDepositRequest` which may not be equal to the `owner`.

MUST be emitted when a deposit cancelation Request is submitted using the `claimCancelDepositRequest` method.

```yaml
- name: CancelDepositClaim
  type: event

  inputs:
    - name: receiver
      indexed: true
      type: address
    - name: owner
      indexed: true
      type: address
    - name: requestId
      indexed: true
      type: uint256
    - name: sender
      indexed: false
      type: address
    - name: assets
      indexed: false
      type: uint256
```

#### CancelRedeemRequest

`owner` has requested cancelation of their deposit Request with request ID `requestId`. `sender` is the caller of the `cancelRedeemRequest` which may not be equal to the `owner`.

MUST be emitted when a deposit cancelation Request is submitted using the `cancelRedeemRequest` method.

```yaml
- name: CancelRedeemRequest
  type: event

  inputs:
    - name: owner
      indexed: true
      type: address
    - name: requestId
      indexed: true
      type: uint256
    - name: sender
      indexed: false
      type: address
```

#### CancelRedeemClaim

`owner` has claimed their redeem cancelation Request with request ID `requestId`. `receiver` is the destination of the `shares`. `sender` is the caller of the `claimCancelRedeemRequest` which may not be equal to the `owner`.

MUST be emitted when a redeem cancelation Request is submitted using the `claimCancelRedeemRequest` method.

```yaml
- name: CancelRedeemClaim
  type: event

  inputs:
    - name: receiver
      indexed: true
      type: address
    - name: owner
      indexed: true
      type: address
    - name: requestId
      indexed: true
      type: uint256
    - name: sender
      indexed: false
      type: address
    - name: shares
      indexed: false
      type: uint256
```

### [ERC-165](./eip-165.md) support

Smart contracts implementing this Vault standard MUST implement the [ERC-165](./eip-165.md) `supportsInterface` function.

Asynchronous deposit Vaults with cancelation support MUST return the constant value `true` if `0x8bf840e3` is passed through the `interfaceID` argument.

Asynchronous redemption Vaults with cancelation support MUST return the constant value `true` if `0xe76cffc7` is passed through the `interfaceID` argument.

## Rationale

## Backwards Compatibility

The interface is fully backwards compatible with [ERC-7540](./eip-7540.md).

## Security Considerations

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
