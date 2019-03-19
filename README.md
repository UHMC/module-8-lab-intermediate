# Module 8 - Intermediate Lab: Fancier Faucet

## Background
This lab will modify a pre-existing faucet smart contract to include additional features.

## Meta Information
| Attribute | Explanation |
| - | - |
| Summary | This assignment involves creating, deploying, and interacting with a faucet smart contract. |
| Topics  | Smart contracts, faucets, tokens. |
| Audience | Appropriate for CS1 or a later course. |
| Difficulty | Intermediate. |
| Strengths | This assignment exposes students to popular technology (blockchain) and some of its features. There is additional focus on some features of the Solidity programming language. |
| Weaknesses | It may be difficult for students to extend smart contracts without programming knowledge and familiarity with the Solidity programming language. |
| Dependencies | Some programming knowledge, an internet-connected computer with a suitable browser (for use of [the Remix IDE][Remix]), and a [MetaMask][MetaMask] account with [Rinkeby test ETH][RinkebyFaucet]. |
| Variants | Could be used to delve deeper into smart contracts, tokens, or use of general-purpose blockchains. |

## Assignment Instructions
1. Navigate to [https://remix.ethereum.org][Remix].
    * This is our development environment. Here, we can write smart contracts and test them.
2. Click on the "+" at the top left corner of the screen.
    * This will create a new empty document to begin development.
3. Name the document `fancier-faucet.sol`.
4. Copy and paste the provided [starter code][StarterCode] into the blank document.
    * This code represents a faucet, which will allow anyone to pay it any amount of ETH and will give anyone freely from its reserve any amount requested, provided that it is less than or equal to 100000000000000000 wei (17 zeros) or 0.1 ETH.
5. Our first modification will be an exercise in modularity and inheritence. In the same file, above the original contract, make a new contract named `Owned`. Then move the `address owner;` line and the `constructor() ... }` block (including the //comment) from the Faucet contract into the Owned contract.
6. Inside the Owned contract, add the following code under the constructor:
    ```solidity
    // Access control modifier
    modifier onlyOwner {
        require(msg.sender == owner, "Only the contract owner can call this function");
        _;
    }
    ```
    * This code creates a _modifier_ which uses the special character `_` to represent any pre-existing code in a block being modified. We'll see how it's used in a moment.
7. Create a new contract below the Owned contract but above the Faucet contract and name it `Mortal`, then move the `// Contract destructor` block into it from the Faucet contract.
8. Delete the `require(...` line, and instead, add `onlyOwner` to the `destroy` function so that it reads `function destroy() public onlyOwner {`.
9. Now we should have a faucet contract that looks something like this:
    ```solidity
    // Version of Solidity compiler this program was written for
    pragma solidity ^0.5.1;
    contract Owned {
        address owner;
        // Contract constructor: set owner
        constructor() public {
            owner = msg.sender;
        }
        // Access control modifier
        modifier onlyOwner {
            require(msg.sender == owner, "Only the contract owner can call this function");
            _;
        }
    }

    contract Mortal is Owned {
    // Contract destructor
        function destroy() public onlyOwner {
            selfdestruct(msg.sender);
        }
    }

    contract Faucet is Mortal {
        event Withdrawal(address indexed to, uint amount);
        event Deposit(address indexed from, uint amount);

        // Give out ether to anyone who asks
        function withdraw(uint withdraw_amount) public {
            // Limit withdrawal amount
            require(withdraw_amount <= 0.1 ether);
            require(address(this).balance >= withdraw_amount, "Insufficient balance in faucet for withdrawal request");
                // Send the amount to the address that requested it
                msg.sender.transfer(withdraw_amount);
                emit Withdrawal(msg.sender, withdraw_amount);
            }
        // Accept any incoming amount
        function () external payable {
            emit Deposit(msg.sender, msg.value);
        }
    }
    ```
10. Alright, it's time to test. Go ahead and click **Start to compile (Ctrl-S)** and see if you get any errors.
11. In the top right, select the **Run** tab. All settings should be the same as for a previous lab, but in the box above the **Deploy** button, it should say **Faucet**. If so, go ahead and click the **Deploy** button.
    * (You will need to confirm the pending transaction in MetaMask, but it should pop up.)
13. In MetaMask, find the most recent transaction. It should say **Contract Deployment** and _confirmed_. Click on that and scroll down. To the right of **Details** you should see an up-and-to-the-right arrow. This will take you to [etherscan][Etherscan]. There you should see (or, if you just clicked on the blue thing in the last sentence, be able to search for (with your address)) the transaction wherein your contract was created.
14. Click on the line that looks like \[Contract 0x... Created\]. That'll take you to a page where the contract is the main focus. At the top of the page, there should be a little copy icon that looks like two pieces of paper, just to the right of the address. Click it. The address of the contract should be copied to your virtual clipboard now.
15. In MetaMask, click on **SEND** and then where it says **Recipient Address**, you'll want to paste (Ctrl-V) the address you just copied a moment ago.
16. Next, set the amount to something reasonable, like 0.5 ETH. Click **NEXT**. If everything looks fine (not 50 ETH), go ahead and click **CONFIRM**. Then just wait for the transaction to go through.
17. Back in [Remix][Remix], within the **Transactions recorded** area, you should see something like **Faucet at 0x... (blockchain)**. Open that up (click on it) and you should see the names of the functions we wrote (`destroy`, `withdraw`...).
18. In the box beside withdraw, enter an amount less than 100000000000000000, and then click withdraw. Once you allow the pending transaction in MetaMask and it goes through, your account should have that much ETH returned to it (minus the transaction fee, which could very well be more than you asked for back).
19. If you want the rest of your ether back (without numerous transaction fees and waiting for every 0.1 ETH increment to go through), you can just click destroy, and when it's done (remember to allow it in MetaMask), your account should be back to its original balance, a little worse for wear...

## Credits
Dr. Debasis Bhattacharya  
Mario Canul  
Saxon Knight  
https://github.com/ethereumbook/ethereumbook  

[Remix]: https://remix.ethereum.org/
[Etherscan]: https://rinkeby.etherscan.io/
[MetaMask]: https://metamask.io/
[RinkebyFaucet]: https://www.rinkeby.io/#faucet
[StarterCode]: https://github.com/UHMC/module-8-lab-intermediate/blob/master/faucet.sol
