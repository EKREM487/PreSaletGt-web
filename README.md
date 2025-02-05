<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ethereum Smart Contract Interaction</title>

    <!-- Content Security Policy to allow Web3.js and Etherscan API -->
    <meta http-equiv="Content-Security-Policy" content="
        default-src 'self'; 
        script-src 'self' https://cdn.jsdelivr.net 'nonce-r4nd0mN0nce';
        connect-src 'self' https://api.etherscan.io;
    ">

    <!-- Web3.js library -->
    <script nonce="r4nd0mN0nce" src="https://cdn.jsdelivr.net/npm/web3@latest/dist/web3.min.js"></script>

    <style>
        body { font-family: Arial, sans-serif; text-align: center; margin: 20px; }
        #accountInfo, #contractInfo { margin-top: 20px; display: none; }
    </style>
</head>
<body>

    <h1>Ethereum Smart Contract Interaction</h1>
    <button id="connectWallet">Connect Wallet</button>

    <div id="accountInfo">
        <h3>Connected Account:</h3>
        <p><strong>Address:</strong> <span id="accountAddress">Not connected</span></p>
        <p><strong>ETH Balance:</strong> <span id="ethBalance">0</span> ETH</p>
    </div>

    <div id="contractInfo">
        <h3>Smart Contract Details</h3>
        <p><strong>Total ETH Raised:</strong> <span id="totalETH">Loading...</span></p>
        <button id="buyTokens">Buy tGt Tokens</button>
        <p id="statusMessage"></p>
    </div>

    <!-- JavaScript with Nonce for Secure Execution -->
    <script nonce="r4nd0mN0nce">
        let web3;
        let contract;
        let userAccount;

        // 📌 Enter your contract address here
        const contractAddress = "0x298Af6b1ac1c846a5E2eB541A4955c4994f3D882";

        // 📌 Manually enter the smart contract ABI here
        const contractABI = [ [{"inputs":[{"internalType":"address","name":"_tokenAddress","type":"address"},{"internalType":"uint256","name":"_startTime","type":"uint256"},{"internalType":"uint256","name":"_endTime","type":"uint256"}],"stateMutability":"nonpayable","type":"constructor"},{"anonymous":false,"inputs":[{"indexed":false,"internalType":"uint256","name":"amountToken","type":"uint256"},{"indexed":false,"internalType":"uint256","name":"amountETH","type":"uint256"}],"name":"LiquidityAdded","type":"event"},{"anonymous":false,"inputs":[{"indexed":false,"internalType":"uint256","name":"totalETH","type":"uint256"},{"indexed":false,"internalType":"uint256","name":"totalTokens","type":"uint256"}],"name":"PresaleEnded","type":"event"},{"anonymous":false,"inputs":[{"indexed":false,"internalType":"uint256","name":"totalRaised","type":"uint256"}],"name":"SoftcapReached","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"internalType":"address","name":"user","type":"address"},{"indexed":false,"internalType":"uint256","name":"amount","type":"uint256"}],"name":"TokensClaimed","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"internalType":"address","name":"buyer","type":"address"},{"indexed":false,"internalType":"uint256","name":"ethAmount","type":"uint256"},{"indexed":false,"internalType":"uint256","name":"tokenAmount","type":"uint256"}],"name":"TokensPurchased","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"internalType":"address","name":"owner","type":"address"},{"indexed":false,"internalType":"uint256","name":"amount","type":"uint256"}],"name":"Withdrawn","type":"event"},{"inputs":[],"name":"buyTokens","outputs":[],"stateMutability":"payable","type":"function"},{"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"claimTimes","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"claimTokens","outputs":[],"stateMutability":"nonpayable","type":"function"},{"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"contributions","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"endTime","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"finalizePresale","outputs":[],"stateMutability":"nonpayable","type":"function"},{"inputs":[],"name":"hardcap","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"liquidityAdded","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"liquidityReserve","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"lockedTokens","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"maxContribution","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"minContribution","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"owner","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"rate","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"softcap","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"startTime","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"token","outputs":[{"internalType":"contract IERC20","name":"","type":"address"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"totalRaised","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"totalTokensForSale","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"uniswapRouter","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"withdrawETH","outputs":[],"stateMutability":"nonpayable","type":"function"},{"inputs":[],"name":"withdrawTokens","outputs":[],"stateMutability":"nonpayable","type":"function"},{"stateMutability":"payable","type":"receive"}] ];

        async function connectWallet() {
            if (typeof window.ethereum !== 'undefined') {
                try {
                    await window.ethereum.request({ method: 'eth_requestAccounts' });
                    web3 = new Web3(window.ethereum);

                    const accounts = await web3.eth.getAccounts();
                    userAccount = accounts[0];

                    document.getElementById('accountAddress').textContent = userAccount;
                    document.getElementById('accountInfo').style.display = 'block';
                    document.getElementById('contractInfo').style.display = 'block';
                    document.getElementById('connectWallet').style.display = 'none';

                    const balance = await web3.eth.getBalance(userAccount);
                    document.getElementById('ethBalance').textContent = web3.utils.fromWei(balance, 'ether');

                    if (contractAddress && contractABI.length > 0) {
                        contract = new web3.eth.Contract(contractABI, contractAddress);
                        updateContractInfo();
                    } else {
                        console.warn("Smart contract address or ABI is missing.");
                    }
                } catch (error) {
                    console.error("Wallet connection error:", error);
                    alert("Failed to connect wallet.");
                }
            } else {
                alert("Please install MetaMask or a Web3-compatible wallet.");
            }
        }

        async function updateContractInfo() {
            if (!contract) return;
            try {
                const totalRaised = await contract.methods.totalRaised().call();
                document.getElementById('totalETH').textContent = web3.utils.fromWei(totalRaised, 'ether') + " ETH";
            } catch (error) {
                console.error("Error fetching smart contract data:", error);
            }
        }

        async function buyTokens() {
            if (!contract) {
                alert("Smart contract is not connected.");
                return;
            }

            try {
                const amount = prompt("Enter the amount of tGt tokens to buy:");
                if (amount <= 0) {
                    alert("Please enter a valid amount.");
                    return;
                }

                const tokenPrice = web3.utils.toWei('0.01', 'ether'); 
                const totalCost = web3.utils.toBN(tokenPrice).mul(web3.utils.toBN(amount));

                document.getElementById('statusMessage').textContent = "Processing transaction...";

                await contract.methods.buyTokens().send({
                    from: userAccount,
                    value: totalCost
                });

                document.getElementById('statusMessage').textContent = "Purchase successful!";
                updateContractInfo();
            } catch (error) {
                console.error("Transaction error:", error);
                document.getElementById('statusMessage').textContent = "Transaction failed.";
            }
        }

        document.getElementById('connectWallet').addEventListener('click', connectWallet);
        document.getElementById('buyTokens').addEventListener('click', buyTokens);
    </script>

</body>
</html>
