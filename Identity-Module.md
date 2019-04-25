Skip to content
 
Search or jump to…

Pull requests
Issues
Marketplace
Explore
 
@thomivy 
7
27 3 hicommonwealth/edgeware-node
 Code  Issues 12  Pull requests 1  Projects 0  Wiki  Insights
edgeware-node
/
modules
/
edge-identity
/
README.md
 

1
# edge_identity
2
The identity module currently handles the registration, attestation, and verification of external identities on Edgeware. The types of identities one may be interested in registering through this module include other blockchain addresses or public keys, Github usernames, email addresses, and even phone numbers. The goal of this module is to be as extensible to external identities in the real or blockchain world to interface with Edgeware.
3
​
4
​
5
# Setup
6
Install rust or update to the latest versions.
7
```
8
curl https://sh.rustup.rs -sSf | sh
9
rustup update nightly
10
rustup target add wasm32-unknown-unknown --toolchain nightly
11
rustup update stable
12
cargo install --git https://github.com/alexcrichton/wasm-gc
13
```
14
​
15
You will also need to install the following packages:
16
​
17
Linux:
18
```
19
sudo apt install cmake pkg-config libssl-dev git clang libclang-dev
20
```
21
​
22
Mac:
23
```
24
brew install cmake pkg-config openssl git llvm
25
```
26
​
27
# Identity Lifecycle
28
​
29
Identities on Edgeware go through a simple state machine:
30
​
31
1. `Registered`
32
2. `Attested`
33
3. `Verified`
34
​
35
## Registration
36
​
37
An external identity on Edgeware first starts at the registration process. A user must possess a valid Edgeware public key and be able to sign and submit a transaction to an eligible Edgeware node. To register an identity, we only require submitting the type of identity and identity formatted as a byte arrays `Vec<u8>` to the `register` function. Once the transaction is included in the Edgeware chain, this identity becomes `Registered`.
38
​
39
## Attested
40
​
41
An attestation of an external identity on Edgeware is a convincing proof that the registrar of the identity actually owns said identity. We denote the registrar as the possessor of the Edgeware private key that submitted the registration of the external identity. If the attester and registrar are the same person, they should have no problem convincing the active set of `verifiers` that they in fact control the external identity. The attestation is formatted as a byte array `Vec<u8>`, which should reliably point to a URL or other satisfiable proof of control.
42
​
43
The sender of both the registration and attestation must be the same. This helps mitigate false attestations. Therefore, the blockchain serves as the single source of truth for attestations on identity registrations. A verifier should only consult the proof stored in the chain to decide whether it is valid or not and not any other form or presentation of an attestation.
44
​
45
Examples of attestations for different settings:
46
​
@thomivy
Commit changes
Commit summary 
Update README.md
Optional extended description
Add an optional extended description…
  Commit directly to the master branch.
  Create a new branch for this commit and start a pull request. Learn more about pull requests.
 
© 2019 GitHub, Inc.
Terms
Privacy
Security
Status
Help
Contact GitHub
Pricing
API
Training
Blog
About
