<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>tGt Presale</title>
    <script src="https://cdn.jsdelivr.net/npm/web3@latest/dist/web3.min.js"></script>
</head>
<body>
    <h2>tGt Token Presale</h2>
    <button onclick="connectWallet()">Connect Wallet</button>
    <p id="walletAddress">Not Connected</p>
    
    <input type="text" id="ethAmount" placeholder="Enter ETH amount">
    <button onclick="buyTokens()">Buy Tokens</button>
    
    <script>
        let web3;
        let contract;
        const contractAddress = "0x951AfD9E496030Fb24a55c55C5a2f2f1520Bdfa5"; // ETH Ana Ağdaki Sözleşme Adresi
        const contractABI = [ /* https://api.etherscan.io/api?module=contract&action=getabi&address=0x951afd9e496030fb24a55c55c5a2f2f1520bdfa5 */ ];

        async function connectWallet() {
            if (window.ethereum) {
                web3 = new Web3(window.ethereum);
                await window.ethereum.request({ method: "eth_requestAccounts" });
                const accounts = await web3.eth.getAccounts();
                document.getElementById("walletAddress").innerText = "Connected: " + accounts[0];

                contract = new web3.eth.Contract(contractABI, contractAddress);
            } else {
                alert("MetaMask is not installed. Please install MetaMask to use this feature.");
            }
        }

        async function buyTokens() {
            const ethAmount = document.getElementById("ethAmount").value;
            if (!ethAmount) {
                alert("Please enter ETH amount.");
                return;
            }

            const accounts = await web3.eth.getAccounts();
            contract.methods.buyTokens().send({
                from: accounts[0],
                value: web3.utils.toWei(ethAmount, "ether")
            })
            .on("transactionHash", function(hash) {
                alert("Transaction sent: " + hash);
            })
            .on("receipt", function(receipt) {
                alert("Transaction confirmed!");
            })
            .on("error", function(error) {
                alert("Transaction failed: " + error.message);
            });
        }
    </script>
</body>
</html>
