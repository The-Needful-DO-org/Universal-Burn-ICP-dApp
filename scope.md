# Universal ICP Token Burn dApp

Week 1 backend

- Function: Add Token to Watchlist, supporting ICRC tokens with an Index canister
- Function: Process tokens on Watchlist
  - Detect balances of received tokens and burn them
  - Add Timer to periodically run the process
  - Add parser to tally and save stats per user

Week 2 backend

- Function: Process tokens on Watchlist
  - Add parser to tally and save stats per user

Week 3 frontend

- Info about dApp
- Social media links
- Instructions (Transfer tokens to backend canister to burn)
- Easy to copy Canister ID / Backend Principal
- Form to Add Token
- Browse by Token
- Browse by User?
- Charts

Week 4 frontend

- Charts

Week 5 frontend

- Charts

I need to think about Frontend a bit more to scope it out in detail.

# Backend

Canister recieves tokens through canister id.
In other words, users transfer tokens to the backend canister id
principal. This is convenient because it can be done from any wallet.

Probably would focus only on ICRC tokens for the initial version,
but I will share some thoughts regarding other standards.

## How are tokens burned?

### ICRC1

ICRC1 tokens are burned by sending them to the Minting Account as
defined by that token ledger function `icrc1_minting_account`.

Some tokens may provide a `burn` function.

Research: Compare the results of `burn` function vs transfer to the
`icrc1_minting_account`. Does the transaction history show both as a
BURN transaction?

### DIP20

DIP20 tokens have a `burn` function which deletes the tokens from both the
caller's balance and the total supply.

### DRC20

I dont know much about drc, need to research.

### ICP

ICP can be burned same as ICRC1 or through the Cycles Minting Canister
which will burn the ICP to mint Cycles.

### NFT

NFTs are burned by assigning no owner. More research needed. Not
planning to support NFTs initially, but sharing some thoughts about
them.

### Concerns & Questions

#### What if a token is non-standard? For example, what if the `burn`
function behaves unexpectedly?

In the interest of progress, I suggest we assume tokens
behave as expected. If problems arise we will deal with it.

#### What if token lacks transaction history?

Seems like it would be impossible to credit the user who sent the
tokens to the Universal Burn Camister.

#### What if user's burn tokens directly?

We could detect burns which happen outside of the Universal Burn
Canister.

## Functions

### Public Function: Add Token

Add a token to the watchlist.

Arguments:

  - Ledger canister id
  - Index canister id

#### Validations

Must be a new token. If token is already in the Watchlist, return an error saying so.

Must be ICRC standard. More standards could be supported later.

Require an index canister.

it is possible to do without an Index canister but this raises the
complexity because the entire transaction history must be parsed.
For this reason I recommend requiring an Index canister and *maybe*
adding support for tokens without an index canister later.

Permissionless, anyone can add tokens.

Data structure of Watchlist could be something like an Array of tokens.

Pseudocode:

    Array<Token>
      Type Token
        { ledger_canister_id: Principal,
          index_canister_id: ?Principal,
          standard: Text (ICRC, DIP20, etc...)
        }

### Private Function: Burn Tokens

Burn the tokens

ICRC send to the minting account (maybe use burn function if ledger
provides?)

DIP20 use burn function

ICP maybe send to minting account, maybe convert to cycles?

DRC20 I dont know much about, need to research.

NFT set owner to null or maybe `aaaaa-aa`

### Private Function: Process recieved tokens

Periodically burn received tokens and update statistics.

Burn stats can be calculated by reading the Index data, specifically
tallying the transfers to the Universal Burn Canister.

It's necessary to store data keeping track of which transfers have
already been processed, and maybe it would be useful to store some other data.

Ideas for data to store:

- Transaction Id of most recently processed transaction per token.
  The point being to efficiently process new transfers to the Universal Burn Canister
- Total amount of each token burned
- Total amounts burned by user
- Individual timestamped burns by user. This data is retrievable from
  the Index, but it might make sense to store it in a manner that is easier
  to display without having to fetch and parse the Index.

#### Concerns

Data not stored could disappear, for example if a token archive and/or
index canister were deleted or suffered data loss. I am not extremely
concerned about this, I am just thinking about the fact this data exists
outside of our control.

Maybe allow the function to be Public with a rate limit, so users could
trigger it instead of having to wait for a timed event.

### Private Function: Remove Token

Maybe should have a function to remove tokens from the watchlist.
If a token hard forks to a new canister, rugs, gets deleted, or whatever
reason to stop watching. Admin coukd use this function and/or maybe if a
token fails to be processed a certain number of times the canister could
be removed from the watchlist or paused.


