# Competitive Gaming Tournament Manager Smart Contract

A comprehensive blockchain-based tournament management system built on the Stacks blockchain using Clarity smart contract language. This contract provides automated tournament organization, participant registration, match scheduling, result tracking, and proportional prize distribution.

## Overview

This smart contract enables decentralized tournament management for competitive gaming with features including:
- Automated tournament lifecycle management
- Secure participant registration with entry fees
- Match scheduling and result submission
- Proportional prize distribution based on performance
- Comprehensive tournament statistics and leaderboards

## Features

### Core Functionality
- **Tournament Creation**: Organizers can create tournaments with customizable parameters
- **Player Registration**: Automatic registration with entry fee handling
- **Match Management**: Schedule matches and submit results
- **Prize Distribution**: Automated proportional prize distribution based on points
- **Tournament States**: Registration → In Progress → Completed lifecycle

### Security Features
- **Access Control**: Administrator and organizer privilege verification
- **Emergency Pause**: Contract-wide pause mechanism for emergency situations
- **Validation**: Comprehensive input validation and state checking
- **Escrow**: Secure handling of entry fees and prize pools

## Contract Structure

### Data Maps
- `tournament-registry`: Core tournament information
- `participant-records`: Player registration and performance data
- `competition-matches`: Match scheduling and results
- `reward-distribution-records`: Prize claim tracking
- `tournament-score-totals`: Tournament-wide scoring
- `tournament-match-counters`: Match ID generation

### Constants
- **Tournament States**: Registration (0), In Progress (1), Completed (2)
- **Scoring**: 3 points per victory, 0 points per defeat
- **Error Codes**: Comprehensive error handling (100-124)

## Public Functions

### Administrative Functions

#### `transfer-administrator-role`
```clarity
(transfer-administrator-role (new-administrator-address principal))
```
Transfer contract administrator privileges to a new address.

#### `set-emergency-pause-state`
```clarity
(set-emergency-pause-state (pause-status bool))
```
Toggle emergency pause to halt all contract operations.

### Tournament Management

#### `establish-new-tournament`
```clarity
(establish-new-tournament 
  (tournament-name (string-ascii 50))
  (tournament-description (string-ascii 255))
  (entry-fee-amount uint)
  (competition-start-block uint)
  (competition-end-block uint)
  (maximum-participant-limit uint))
```
Create a new tournament with specified parameters.

**Parameters:**
- `tournament-name`: Tournament display name (max 50 characters)
- `tournament-description`: Tournament description (max 255 characters)
- `entry-fee-amount`: Entry fee in microSTX (0 for free tournaments)
- `competition-start-block`: Block height when tournament starts
- `competition-end-block`: Block height when tournament ends
- `maximum-participant-limit`: Maximum number of participants

**Returns:** Tournament ID on success

#### `register-tournament-participant`
```clarity
(register-tournament-participant (tournament-identifier uint))
```
Register the calling address as a tournament participant.

**Requirements:**
- Tournament must be in registration state
- Player must not be already registered
- Entry fee must be paid if required
- Tournament must not be at capacity

#### `commence-tournament-competition`
```clarity
(commence-tournament-competition (tournament-identifier uint))
```
Transition tournament from registration to active competition phase.

**Requirements:**
- Must be called by administrator or tournament organizer
- Tournament must be in registration state
- Minimum 2 participants required

#### `finalize-tournament-completion`
```clarity
(finalize-tournament-completion (tournament-identifier uint))
```
Mark tournament as completed and enable prize distribution.

**Requirements:**
- Must be called by administrator or tournament organizer
- Tournament must be in progress
- Competition end block must be reached

### Match Management

#### `schedule-tournament-match`
```clarity
(schedule-tournament-match 
  (tournament-identifier uint) 
  (first-competitor principal) 
  (second-competitor principal) 
  (tournament-round uint))
```
Schedule a match between two tournament participants.

**Requirements:**
- Must be called by administrator or tournament organizer
- Tournament must be in active state
- Both competitors must be registered participants

**Returns:** Match ID on success

#### `submit-match-outcome`
```clarity
(submit-match-outcome 
  (tournament-identifier uint) 
  (match-identifier uint) 
  (declared-winner principal))
```
Submit the result of a completed match.

**Requirements:**
- Must be called by administrator or tournament organizer
- Match must exist and not have a result already
- Winner must be one of the two competitors
- Tournament must be in active state

### Prize Distribution

#### `process-prize-claim`
```clarity
(process-prize-claim (tournament-identifier uint))
```
Claim proportional prize money based on tournament performance.

**Requirements:**
- Tournament must be completed
- Participant must not have already claimed
- Participant must have earned points (prize > 0)

**Returns:** Prize amount claimed

## Read-Only Functions

### Query Functions

#### `get-tournament-details`
```clarity
(get-tournament-details (tournament-identifier uint))
```
Retrieve complete tournament information.

#### `get-participant-profile`
```clarity
(get-participant-profile (tournament-identifier uint) (participant-address principal))
```
Get participant registration and performance statistics.

#### `get-match-details`
```clarity
(get-match-details (tournament-identifier uint) (match-identifier uint))
```
Retrieve match information and results.

#### `calculate-participant-prize-share`
```clarity
(calculate-participant-prize-share (tournament-identifier uint) (participant-address principal))
```
Calculate proportional prize amount for a participant.

#### `get-participant-match-history`
```clarity
(get-participant-match-history (tournament-identifier uint) (participant-address principal))
```
Get list of match IDs involving the specified participant.

## Usage Example

### Creating a Tournament
```clarity
;; Create a tournament with 100 STX entry fee
(contract-call? .tournament-contract establish-new-tournament
  "Summer Championship"
  "Annual summer gaming tournament with prizes"
  u100000000  ;; 100 STX in microSTX
  u1000       ;; Start block
  u2000       ;; End block
  u16)        ;; Max 16 participants
```

### Registering for a Tournament
```clarity
;; Register for tournament ID 1
(contract-call? .tournament-contract register-tournament-participant u1)
```

### Starting a Tournament
```clarity
;; Start tournament (organizer only)
(contract-call? .tournament-contract commence-tournament-competition u1)
```

### Scheduling a Match
```clarity
;; Schedule match between two players
(contract-call? .tournament-contract schedule-tournament-match 
  u1                    ;; Tournament ID
  'ST1PLAYER1ADDRESS    ;; First competitor
  'ST2PLAYER2ADDRESS    ;; Second competitor
  u1)                   ;; Round number
```

### Submitting Match Results
```clarity
;; Submit match result
(contract-call? .tournament-contract submit-match-outcome
  u1                    ;; Tournament ID
  u0                    ;; Match ID
  'ST1PLAYER1ADDRESS)   ;; Winner
```

### Claiming Prizes
```clarity
;; Claim tournament prize
(contract-call? .tournament-contract process-prize-claim u1)
```

## Error Codes

| Code | Error | Description |
|------|-------|-------------|
| 100 | ERR-UNAUTHORIZED-ACCESS | Caller lacks required permissions |
| 101 | ERR-TOURNAMENT-DOES-NOT-EXIST | Tournament ID not found |
| 102 | ERR-PLAYER-ALREADY-REGISTERED | Player already registered for tournament |
| 103 | ERR-REGISTRATION-PERIOD-CLOSED | Registration phase has ended |
| 104 | ERR-MATCH-RECORD-NOT-FOUND | Match ID not found |
| 105 | ERR-MATCH-RESULT-ALREADY-SUBMITTED | Match result already recorded |
| 106 | ERR-TOURNAMENT-CURRENTLY-ACTIVE | Tournament is already active |
| 107 | ERR-TOURNAMENT-NOT-IN-ACTIVE-STATE | Tournament not in active state |
| 108 | ERR-INVALID-PARTICIPANT | Participant not registered |
| 109 | ERR-INSUFFICIENT-FUNDS-BALANCE | Insufficient STX balance |
| 110 | ERR-PRIZE-MONEY-ALREADY-CLAIMED | Prize already claimed |
| 111 | ERR-NOT-ELIGIBLE-FOR-PRIZE | No prize eligibility |
| 112 | ERR-INVALID-TOURNAMENT-PHASE | Invalid tournament state |
| 113 | ERR-TOURNAMENT-HAS-CONCLUDED | Tournament already concluded |
| 114 | ERR-CONTRACT-OPERATIONS-PAUSED | Contract is paused |
| 115 | ERR-TOURNAMENT-NOT-CONCLUDED | Tournament not yet concluded |
| 120 | ERR-INVALID-TIME-PARAMETERS | Invalid time parameters |
| 121 | ERR-START-BLOCK-IN-PAST | Start block is in the past |
| 122 | ERR-INSUFFICIENT-PLAYER-CAPACITY | Player capacity too low |
| 123 | ERR-MINIMUM-PLAYERS-NOT-MET | Minimum players not met |
| 124 | ERR-END-BLOCK-NOT-REACHED | End block not yet reached |

## Tournament Lifecycle

1. **Creation**: Organizer creates tournament with parameters
2. **Registration**: Players register and pay entry fees
3. **Competition**: Tournament begins, matches are scheduled and results submitted
4. **Completion**: Tournament ends, final standings calculated
5. **Prize Distribution**: Participants claim proportional prizes

## Prize Distribution

Prizes are distributed proportionally based on points earned:
- **Points System**: 3 points per victory, 0 points per defeat
- **Calculation**: `(participant_points / total_tournament_points) × prize_pool`
- **Automatic Distribution**: Participants claim their earned prizes

## Security Considerations

- **Access Control**: Functions restricted to appropriate roles
- **State Validation**: Comprehensive state checking before operations
- **Emergency Controls**: Pause mechanism for critical situations
- **Escrow Protection**: Entry fees held securely until distribution