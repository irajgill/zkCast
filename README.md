## Overview 
zkCast is a zero-knowledge (ZK) voting system implementation in Circom. This project provides a simple, privacy-preserving voting protocol using ZK proofs, specifically focusing on anonymous vote selection. The included Circom code demonstrates how a user's vote can be committed without revealing their choice, yet still allows verification that the vote is valid and belongs to a specific candidate index.


## Setup Instructions

### Prerequisites
- **Circom** – Circuits compiler. Install from the official `iden3/circom` repository[1].
- **snarkjs** – Command-line toolkit for proof generation and verification.
- **Node.js** – JavaScript runtime required by `snarkjs` scripts.
- **Rust & Cargo** – Needed only when compiling Circom from source instead of using pre-built binaries.

## Build and Run
`git clone https://github.com/irajgill/zkCast.git`
`cd zkCast/circom`

Install Circom

Follow Circom installation instructions.

Install snarkjs
`npm install -g snarkjs`

## How It Works

The main circuit in `zkCast` enables anonymous voting using zero-knowledge (ZK) proofs. By leveraging cryptographic commitments and ZK-friendly hash functions (specifically, Poseidon), voters can securely cast ballots without revealing which candidate they chose. The circuit enforces the following constraints:
- The vote commitment equals the hash of the voter's secret and their selected candidate index.
- The chosen candidate index falls within an allowed range.
- Only one candidate is chosen per vote (one-hot selection).
- All protocol constraints are satisfied for a valid proof.

## Circom Circuit Explanation

### Files

| File               | Purpose                                                                 |
|--------------------|-------------------------------------------------------------------------|
| `zkCast.circom`    | Core template implementing private voting logic using ZK principles.     |
|                    | - Uses Poseidon Hash (from circomlib) for privacy-preserving commitments|
|                    | - Implements one-hot selection and valid index checks                   |
| `zkCast.sym`       | Auto-generated symbol file for debugging and proof analysis              |

### Core Logic Outline

- **Private Inputs:**
  - `voter_secret`: The private value unique to the voter.
  - `vote_commitment`: A cryptographic commitment (hash) created from the secret and candidate index.
  - `selected_candidate_index`: The 0-based index of the chosen candidate.

- **Public Inputs:**
  - `candidates`: An array of identifiers for each candidate.

- **Outputs:**
  - `vote_valid`: Confirms that all protocol constraints are satisfied and the vote is valid.

#### Workflow

1. Hash `(voter_secret, selected_candidate_index)` using Poseidon and compare the result to `vote_commitment`.
2. Verify that `selected_candidate_index` is within the bounds of the `candidates` array.
3. Enforce a one-hot encoding, ensuring only one candidate is selected per vote.

## Build and Testing

1. Compile the Circuit: `circom zkCast.circom --r1cs --wasm --sym`

2. Run Trusted Setup(Powers of Tau Caremony):

`snarkjs powersoftau new bn128 12 pot12_0000.ptau -v`
`snarkjs powersoftau contribute pot12_0000.ptau pot12_0001.ptau --name="First contribution" -v`
`snarkjs powersoftau prepare phase2 pot12_0001.ptau pot12_final.ptau -v`

3. Generate Proving and Verification Keys:

`snarkjs groth16 setup zkCast.r1cs pot12_final.ptau circuit_0000.zkey`
`snarkjs zkey contribute circuit_0000.zkey circuit_final.zkey --name="Second contribution" -v`
`snarkjs zkey export verificationkey circuit_final.zkey verification_key.json`

4. Prepare Inputs:

Create an `input.json` file with the appropriate values for `candidates`, `voter_secret`, `vote_commitment`, and `selected_candidate_index`.

5. Generate Witness and Proof:

`node zkCast_js/generate_witness.js zkCast_js/zkCast.wasm input.json witness.wtns`
`snarkjs groth16 prove circuit_final.zkey witness.wtns proof.json public.json`

6. Verify Proof:

`snarkjs groth16 verify verification_key.json public.json proof.json`






