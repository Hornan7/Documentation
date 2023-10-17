## The SanchoNet Governance State example
This directory is made temporarily to give an example of the governance State on the Sancho net using the command:
```
cardano-cli conway governance query gov-state \
--testnet-magic 4 \
--out-file governanceState.json
```

#### Show governance actions that will expire at the end of the current epoch:

```
current_epoch=$(cardano-cli query tip --testnet-magic 4 | jq .epoch)
```
```
cardano-cli conway governance query gov-state --testnet-magic 4 \
    | jq --argjson epoch "$current_epoch" '.gov.curGovSnapshots.psGovActionStates 
    | to_entries[] 
    | select(.value.expiresAfter == $epoch)'
```

#### Show governance actions that were proposed in the current epoch:  

```
current_epoch=$(cardano-cli query tip --testnet-magic 4 | jq .epoch)
```
```
cardano-cli conway governance query gov-state --testnet-magic 4 \
    | jq -r --argjson epoch "$current_epoch" '.gov.curGovSnapshots.psGovActionStates 
    | to_entries[] 
    | select(.value.proposedIn == $epoch)'
```

#### Sort governance actions by number of DRep votes: 

```
cardano-cli conway governance query gov-state --testnet-magic 4 | jq -r '
  .gov.curGovSnapshots.psGovActionStates
  | to_entries[]
  | {govActionId: .key, type: .value.action.tag, drepVoteCount: (.value.dRepVotes | keys | length)}
  ' | jq -s 'sort_by(.voteCount) | reverse[]'
```

#### Sort by number of SPO votes:

```
cardano-cli conway governance query gov-state --testnet-magic 4 | jq -r '
  .gov.curGovSnapshots.psGovActionStates
  | to_entries[]
  | {govActionId: .key, type: .value.action.tag, spoVoteCount: (.value.stakePoolVotes | keys | length)}
  ' | jq -s 'sort_by(.voteCount) | reverse[]'
```

#### Sort by number of CC votes:

```
cardano-cli conway governance query gov-state --testnet-magic 4 | jq -r '
  .gov.curGovSnapshots.psGovActionStates
  | to_entries[]
  | {govActionId: .key, ccVoteCount: (.value.committeeVotes | keys | length)}
  ' | jq -s 'sort_by(.voteCount) | reverse[]'
```

#### Filter actions that expire within the current and next 2 epochs, include information about all roles votes, sort by expiration epoch:

```
current_epoch=$(cardano-cli query tip --testnet-magic 4 | jq .epoch)
```

```
cardano-cli conway governance query gov-state --testnet-magic 4 | jq -r --argjson current_epoch "$current_epoch" '
  .gov.curGovSnapshots.psGovActionStates
  | to_entries[]
  | select(.value.expiresAfter >= ($current_epoch | tonumber) and .value.expiresAfter <= ($current_epoch + 2))
  | {
      govActionId: .key,
      type: .value.action.tag,
      expiresAfter: .value.expiresAfter,
      committeeVotesCount: (.value.committeeVotes | length),
      dRepVotesCount: (.value.dRepVotes | length),
      stakePoolVotesCount: (.value.stakePoolVotes | length)
    }
  ' | jq -s 'sort_by(.expiresAfter)'
```

#### Show those actions where a given drep key has already voted, how the drep voted and show how many votes the aciton has:

Replace `058b60ead63f667c0ff5b40e269dd1f05ce3a804256735ad4eddce20` with the hex drep id of your interest.

```
cardano-cli conway governance query gov-state --testnet-magic 4 | jq -r --arg dRepKey "keyHash-058b60ead63f667c0ff5b40e269dd1f05ce3a804256735ad4eddce20" '
  .gov.curGovSnapshots.psGovActionStates
  | to_entries[]
  | select(.value.dRepVotes[$dRepKey] != null)
  | {
      govActionId: .key,
      type: .value.action.tag,
      dRepVote: .value.dRepVotes[$dRepKey],
      expiresAfter: .value.expiresAfter,
      committeeVotesCount: (.value.committeeVotes | length),
      dRepVotesCount: (.value.dRepVotes | length),
      stakePoolVotesCount: (.value.stakePoolVotes | length)
    }
  '
```

#### Show the actions where the given drep key has not voted yet:

Replace `058b60ead63f667c0ff5b40e269dd1f05ce3a804256735ad4eddce20` with the hex drep id of your interest.

```
cardano-cli conway governance query gov-state --testnet-magic 4 | jq -r --arg dRepKey "keyHash-058b60ead63f667c0ff5b40e269dd1f05ce3a804256735ad4eddce20" '
  .gov.curGovSnapshots.psGovActionStates
  | to_entries[]
  | select(.value.dRepVotes[$dRepKey] == null)
  | {
      govActionId: .key,
      type: .value.action.tag,
      expiresAfter: .value.expiresAfter,
      committeeVotesCount: (.value.committeeVotes | length),
      dRepVotesCount: (.value.dRepVotes | length),
      stakePoolVotesCount: (.value.stakePoolVotes | length)
    }
  '
```

#### Show the total number of yes, no and abstain votes for a given governance action id:

Replace `cf2dce795cef4f8e92f0ab062a5dae0c1f7d8891d943ced9c2eeda9a62d8f092#0` with the governance action ID of your interest.

```
cardano-cli conway governance query gov-state --testnet-magic 4 | jq -r --arg actionId "cf2dce795cef4f8e92f0ab062a5dae0c1f7d8891d943ced9c2eeda9a62d8f092#0" '
  .gov.curGovSnapshots.psGovActionStates
  | to_entries[]
  | select(.key == $actionId)
  | { govActionId: .key,
      dRepVoteYesCount: (.value.dRepVotes | with_entries(select(.value == "VoteYes")) | length),
      dRepVoteNoCount: (.value.dRepVotes | with_entries(select(.value == "VoteNo")) | length),
      dRepAbstainCount: (.value.dRepVotes | with_entries(select(.value == "Abstain")) | length),
      stakePoolVoteYesCount: (.value.stakePoolVotes | with_entries(select(.value == "VoteYes")) | length),
      stakePoolVoteNoCount: (.value.stakePoolVotes | with_entries(select(.value == "VoteNo")) | length),
      stakePoolAbstainCount: (.value.stakePoolVotes | with_entries(select(.value == "Abstain")) | length),
      committeeVoteYesCount: (.value.committeeVotes | with_entries(select(.value == "VoteYes")) | length),
      committeeVoteNoCount: (.value.committeeVotes | with_entries(select(.value == "VoteNo")) | length),
      committeeAbstainCount: (.value.committeeVotes | with_entries(select(.value == "Abstain")) | length)
    }
  '
```

#### Show he active `treasury withdrawal` governance actions and their current vote count:

```
current_epoch=$(cardano-cli query tip --testnet-magic 4 | jq .epoch)
cardano-cli conway governance query gov-state --testnet-magic 4 | jq -r --arg currentEpoch "$current_epoch" '
  .gov.curGovSnapshots.psGovActionStates
  | to_entries[]
  | select(.value.expiresAfter > ($currentEpoch | tonumber) and .value.action.tag == "TreasuryWithdrawals")
  | { govActionId: .key,
      type: .value.action.tag,
      expiresAfter: .value.expiresAfter,
      dRepVoteYesCount: (.value.dRepVotes | with_entries(select(.value == "VoteYes")) | length),
      dRepVoteNoCount: (.value.dRepVotes | with_entries(select(.value == "VoteNo")) | length),
      dRepAbstainCount: (.value.dRepVotes | with_entries(select(.value == "Abstain")) | length),
      committeeVoteYesCount: (.value.committeeVotes | with_entries(select(.value == "VoteYes")) | length),
      committeeVoteNoCount: (.value.committeeVotes | with_entries(select(.value == "VoteNo")) | length),
      committeeAbstainCount: (.value.committeeVotes | with_entries(select(.value == "Abstain")) | length)
    }
' | jq -s 'sort_by(.expiresAfter)'
```

#### Show the active `update committee` governance actions and their current vote count: 

```
current_epoch=$(cardano-cli query tip --testnet-magic 4 | jq .epoch)
cardano-cli conway governance query gov-state --testnet-magic 4 | jq -r --arg currentEpoch "$current_epoch" '
  .gov.curGovSnapshots.psGovActionStates
  | to_entries[]
  | select(.value.expiresAfter > ($currentEpoch | tonumber) and .value.action.tag == "UpdateCommittee")
  | { govActionId: .key,
      type: .value.action.tag,
      expiresAfter: .value.expiresAfter,
      dRepVoteYesCount: (.value.dRepVotes | with_entries(select(.value == "VoteYes")) | length),
      dRepVoteNoCount: (.value.dRepVotes | with_entries(select(.value == "VoteNo")) | length),
      dRepAbstainCount: (.value.dRepVotes | with_entries(select(.value == "Abstain")) | length),
      stakePoolVoteYesCount: (.value.stakePoolVotes | with_entries(select(.value == "VoteYes")) | length),
      stakePoolVoteNoCount: (.value.stakePoolVotes | with_entries(select(.value == "VoteNo")) | length),
      stakePoolAbstainCount: (.value.stakePoolVotes | with_entries(select(.value == "Abstain")) | length)
    }
' | jq -s 'sort_by(.expiresAfter)'
```
