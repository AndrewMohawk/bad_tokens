# Bad Token Exploit Repository

## Overview
This repository serves as an archive of documented exploits for various tokens across multiple blockchain networks. Each token exploit is organized by the blockchain and token identifier in a structured format to facilitate easy navigation and accessibility.

## Repository Structure
Each token exploit is stored in a dedicated folder named according to the blockchain and token symbol it pertains to. The format for these folders is `<chain>_<token>`, where:

- `<chain>` is the abbreviation for the blockchain network (e.g., `bsc` for Binance Smart Chain).
- `<token>` is the symbol of the token (e.g., `SATX` for the SATX token).

### Example Directory Name
`eth_PonziToken` - This folder contains exploits related to the PonziToken on the Ethereum network.

## Contents
Each token-specific folder contains the following:

- `README.md`: A detailed overview of the token, the nature of the exploit(s), and any known fixes or mitigation strategies.
- `*.sol`: Relevant smart contract code and snippets that demonstrate the exploit.
- Additional resources such as images, diagrams, and external links to advisories or reports.

## Current Tokens Documented
Below is a list of all tokens for which exploits have been documented in this repository. This list is regularly updated as new exploits are discovered and analyzed.

- **Binance Smart Chain (BSC)**
  - [SATX](./bsc_SATX) - Documentation and analysis of exploits related to the SATX token on Binance Smart Chain.

## Adding New Token Exploits
To add a new token exploit:

1. Create a new folder in the root of the repository using the naming convention `<chain>_<token>`.
2. Include a `README.md` in the folder to provide an overview and detailed analysis of the exploit.
3. Add any additional files such as contract snippets (`contract_snippets.sol`) or detailed explanations (`exploit_details.md`).

## Contribution
Contributions to this repository are welcome. To contribute:

- **Fork** the repository.
- **Create a Feature branch**: `git checkout -b feature-YourFeature`
- **Commit** your changes.
- **Push** to the branch: `git push origin feature-YourFeature`
- **Open a Pull Request**.

## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Contact
For any queries or further assistance, please contact [`@AndrewMohawk`].