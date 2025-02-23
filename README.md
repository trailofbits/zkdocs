# ZKDocs
ZKDocs provides comprehensive, detailed, and interactive documentation on zero-knowledge proof systems and related primitives.

At [Trail of Bits](https://www.trailofbits.com/), we audit many implementations of non-standardized cryptographic protocols and often find the same issues. As we discovered more instances of these bugs, we wanted to find a way to prevent them in the future. Unfortunately, for these protocols, the burden is on the developers to figure out all of the low-level implementation details and security pitfalls.

We hope that ZKDocs can fill in this gap and benefit the larger cryptography community.

## Key Features

### Comprehensive
We aim to be both self-contained and comprehensive in the topics related to zero-knowledge proof systems, from descriptions of simple systems like Schnorr's identification protocol, to complex proof systems like Paillier-Blum modulus. We also cover cryptographic primitives such as: random sampling, Fiat-Shamir transformation, and Shamir's Secret Sharing.

### Detailed
We describe each protocol in great detail, including all necessary setup, sanity-checks, auxiliary algorithms, further references, and potential security pitfalls with their associated severity.

### Interactive
The protocol descriptions are interactive, letting you modify variable names. This allows you to match the variable names in ZKdocs' specification to the variable names in your code, making it easier to find bugs and missing assertions.

![Basic interactivity usage](/static/figs/demo.gif)

#### Interactivity features:
 - Click on a variable to highlight it across the document
 - Type or paste with a variable highlighted to edit its name. Press `Enter` or `Escape` to stop editing
 - Press the `Reset variable names` button to reset the names of all variables on the current page (variable names are independent across different pages)

## Getting Started

### Prerequisites
- [Hugo](https://gohugo.io/documentation/) (v0.80.0 or later)
  - macOS: `brew install hugo`
  - Windows: `choco install hugo-extended`
  - Linux: `sudo apt install hugo`

### Local Development
1. Clone the repository:
```bash
git clone https://github.com/trailofbits/zkdocs.git
cd zkdocs
```

2. Start the development server:
```bash
hugo server --minify --theme book
```

3. Open your browser and navigate to `http://localhost:1313`

### Testing
Before submitting your changes, make sure to:
1. Run the development server and verify your changes locally
2. Check that all mathematical formulas render correctly
3. Test the interactive features with your modifications

----

## Roadmap

### Zero-knowledge proof systems
 - [x] Schnorr basic identification protocol
 - [x] Schnorr variants
 - [x] Product of primes
 - [x] Square-free zkp
 - [x] Short proofs for factoring
 - [x] Girault's identification
 - [x] Paillier-Blum Modulus ZK
 - [ ] Discrete log equality
 - [ ] Ring-Pedersen Parameters ZK
 - [ ] STARK
 - [ ] Paillier range proofs
 - [ ] Bulletproofs
 - [ ] Sonic
 - [ ] Plonk

### Primitives
 - [x] Fiat-Shamir transformation
 - [x] Rejection sampling
 - [x] Nothing-up-my-sleeve constructions
 - [x] Shamir secret sharing
 - [x] Feldman's VSS
 - [ ] Fujisaki-Okamoto commitments
 - [ ] Pedersen commitments
 - [ ] HVZK and NIZK

### Common attacks and issues
 - [x] Using HVZKP in the wrong context: two attacks when verifiers are not so honest
 - [ ] Golden-shoe attack
 - [ ] Alpha-rays: attacking others by having short keys
 - [ ] Replay attacks on ZKPs

----

## Dependencies
 - [hugo](https://gohugo.io/documentation/) - install with

    `brew install hugo`

## Running locally
 - `hugo server --minify --theme book`

## How to contribute
 - The file [schnorr.md](/content/docs/zkdocs/zero-knowledge-protocols/schnorr.md) is an example of a complete protocol.
 - [interactive_variables.js](static/js/interactive_variables.js) has all the variable renaming logic.
 - The Sigma protocols are structured in latex in 3 columns: Alice column, arrow_column, Verifier column. To write the protocols, you can use helpful latex macros:
   - `\work{Work for Alice}{Work for Bob}` - writes work in both Alice's and Bob's column
   - `\alicework{Work for Alice}`, `\bobwork{Work for Bob}` - writes work for either Alice or Bob
   - `\alicebob{Alice work}{message description}{Bob work}`, `\bobalice{Alice work}{message description}{Bob work}` - writes an arrow from alice to bob, or from bob to alice
   - In markdown you would write
```latex
{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\samplezqs{\varr}}
 \alicework{\varu = \varg^\varr}
 \alicebob{}{\varu}{}
 \bobwork{\sample{\varc}}
 \bobalice{}{\varc}{}
 \alicework{\varz = \varr + \varx\cdot \varc}
 \alicebob{}{\varz}{}
 \bobwork{\varg^{\varz} \equalQ \varu \cdot \varh^\varc }
 \end{array}
 $$
{{< /rawhtml >}}
```
 - [header.html](themes/book/layouts/partials/docs/header.html) has all latex macros if more need to be added. In particular it includes all interactive variable macros that the javascript handles afterwards. So, if you write `$\varz$` it will default to a `z` but the user can change its name anywhere on the page.

## Contributing Guidelines

### How to Contribute
1. Fork the repository
2. Create a new branch for your feature
3. Make your changes
4. Submit a pull request

### Commit Message Format
Please follow these guidelines for commit messages:
- Use the present tense ("Add feature" not "Added feature")
- Use the imperative mood ("Move cursor to..." not "Moves cursor to...")
- Limit the first line to 72 characters or less
- Reference issues and pull requests liberally after the first line

### Documentation Style
- The file [schnorr.md](/content/docs/zkdocs/zero-knowledge-protocols/schnorr.md) serves as a template for protocol documentation
- Use LaTeX macros for consistent protocol representation
- Follow the existing structure for new protocol additions

## License
This project is licensed under the MIT License - see the LICENSE file for details

## Contact
For questions and support, please open an issue in the GitHub repository.
