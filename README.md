<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>tGt Token Pre-Sale</title>
    <script src="https://cdn.jsdelivr.net/npm/web3@latest/dist/web3.min.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            text-align: center;
            background: #f4f4f4;
        }
        .container {
            max-width: 600px;
            margin: auto;
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }
        button {
            padding: 10px 20px;
            margin: 10px;
            border: none;
            cursor: pointer;
            font-size: 16px;
            border-radius: 5px;
        }
        .connect-btn {
            background: #28a745;
            color: white;
        }
        .buy-btn {
            background: #007bff;
            color: white;
        }
        .loading {
            font-size: 14px;
            color: #888;
            display: none;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>tGt Token Pre-Sale</h1>
    <button id="connectWallet" class="connect-btn">Connect Wallet</button>
    
    <div id="accountInfo" style="display: none;">
        <p><strong>Wallet:</strong> <span id="walletAddress"></span></p>
        <p><strong>ETH Balance:</strong> <span id="ethBalance"></span> ETH</p>
    </div>

    <div id="preSaleData" style="display: none;">
        <h2>Pre-Sale Details</h2>
        <p><strong>Start Time:</strong> <span id="startTime"></span></p>
        <p><strong>Total ETH Raised:</strong> <span id="totalRaised"></span> ETH</p>
        <p><strong>Remaining TGt Tokens:</strong> <span id="remainingTokens"></span></p>
        <div class="loading" id="loadingMessage">Fetching data...</div>
        
        <h3>Buy TGt Tokens</h3>
        <input type="number" id="amount" min="1" step="1" placeholder="Enter amount">
        <button id="buyButton" class="buy-btn">Buy Tokens</button>
        <p id="statusMessage"></p>
    </div>
</div>

<script>
    // Smart Contract Information
    const contractAddress = "0x3b6F824563Dcd221d4734D11Ec5c3D5848302Eaa";
    const contractABI = [/* [{"inputs":[],"stateMutability":"ödenemez","type":"constructor"},{"anonymous":false,"inputs":[{"indexed":true,"internalType":"address","name":"owner","type":"address"},{"indexed":true,"internalType":"address","name":"spender","type":"address"},{"indexed":false,"internalType":"uint256","name":"value","type":"uint256"}],"name":"Onay","type":"event"},{"anonymous":false,"in puts":[{"indexed":true,"internalType":"adres","name":"from","type":"address"},{"indexed":true,"internalType":"adres","name":"to","type":"address"},{"indexed":false,"internalType":"uint256","name":"value","type":"uint256"}],"name":"Transfer","type":"event"},{"inputs":[{"internalType":"adres","name":"","type":"address"},{"internalType":"adres","name":"","type":"address"} ],"name":"izin","çıktılar":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"görünüm","type":"işlev"},{"inputs":[{"internalType":"adres","name":"harcama","type":"adres"},{"internalType":"uint256","name":"değer","type":"uint256"}],"name":"onayla","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"ödenemez","type":" işlev"},{"girişler":[{"internalType":"adres","ad":"sahip","tür":"adres"}],"ad":"bakiye","çıktılar":[{"internalType":"uint256","ad":"","tür":"uint256"}],"durumDeğişebilirliği":"görünüm","tür":"işlev"},{"girişler":[{"internalType":"adres","ad":"","tür":"adres"}],"ad":"bakiyeler","çıktılar":[{"internalType":"uint256","ad":"","tür":"uint256"}],"durumDeğişebilirliği":"görünüm","tür":"işlev"},{"girişler":[],"ad":"ondalıklar","çıktılar":[{"dahiliTür":"uint256","ad":"","tür":"uint256"}],"durumDeğişebilirliği":"görünüm","tür":"işlev"},{"girişler":[],"ad":"ad","çıktılar":[{"dahiliTür":"dize","ad":"","tür":"dize"}],"durumDeğişebilirliği":"görünüm","tür ":"işlev"},{"girişler":[],"ad":"sembol","çıktılar":[{"dahiliTür":"dize","ad":"","tür":"dize"}],"durumDeğişebilirliği":"görünüm","tür":"işlev"},{"girişler":[],"ad":"toplamTedarik","çıktılar":[{"dahiliTür":"uint256","ad":"","tür":"uint256"}],"durumDeğişebilirliği":"görünüm","tür":"işlev iyon"},{"girişler":[{"internalType":"adres","ad":"kime","tür":"adres"},{"internalType":"uint256","ad":"değer","tür":"uint256"}],"ad":"aktarma","çıktılar":[{"internalType":"bool","ad":"","tür":"bool"}],"durumDeğişebilirliği":"ödenemez","tür":"işlev"},{"girişler":[{"internalType":"adres","ad":"kime","tür":"adres"},{"internalType":"adres","ad":"kime","tür":"adres"},{"internalType":"uint256","ad":"değer","tür":"uint256"}],"ad":"aktarmaFrom","çıktılar":[{"internalType":"bool","ad":"","tür":"bool"}],"durumDeğişebilirliği":"ödenemez","tür":"işlev"}]"çıktılar":[{"internalType":"dize","adı":"","türü":"dize"}],"durumDeğişebilirliği":"görünüm","türü":"işlev"},{"girişler":[],"ad":"toplamTedarik","çıktılar":[{"internalType":"uint256","ad":"","türü":"uint256"}],"durumDeğişebilirliği":"görünüm","türü":"işlev"},{"girişler":[{"internalType":"adres","türü":"kime","türü":"adres"},{"internalType":"uint256","ad":"değer","türü":"uint256"}],"ad":"aktarım","çıktılar":[{"içinde ternalType":"bool","name":"","type":"bool"}],"stateMutability":"ödenemez","type":"işlev"},{"girişler":[{"internalType":"adres","name":"kaynak","type":"adres"},{"internalType":"adres","name":"hedef","type":"adres"},{"internalType":"uint256","name":"değer","type":"uint256"}],"name":"transferFrom","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"ödenemez","type":"işlev"}]"çıktılar":[{"internalType":"dize","adı":"","türü":"dize"}],"durumDeğişebilirliği":"görünüm","türü":"işlev"},{"girişler":[],"ad":"toplamTedarik","çıktılar":[{"internalType":"uint256","ad":"","türü":"uint256"}],"durumDeğişebilirliği":"görünüm","türü":"işlev"},{"girişler":[{"internalType":"adres","türü":"kime","türü":"adres"},{"internalType":"uint256","ad":"değer","türü":"uint256"}],"ad":"aktarım","çıktılar":[{"içinde ternalType":"bool","name":"","type":"bool"}],"stateMutability":"ödenemez","type":"işlev"},{"girişler":[{"internalType":"adres","name":"kaynak","type":"adres"},{"internalType":"adres","name":"hedef","type":"adres"},{"internalType":"uint256","name":"değer","type":"uint256"}],"name":"transferFrom","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"ödenemez","type":"işlev"}]"type":"uint256"}],"name":"transferFrom","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"ödenemez","type":"function"}]"type":"uint256"}],"name":"transferFrom","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"ödenemez","type":"function"}] */];

    let web3;
    let contract;

    document.getElementById("connectWallet").addEventListener("click", async () => {
        if (window.ethereum) {
            web3 = new Web3(window.ethereum);
            try {
                await window.ethereum.request({ method: "eth_requestAccounts" });
                const accounts = await web3.eth.getAccounts();
                document.getElementById("walletAddress").textContent = accounts[0];
                document.getElementById("accountInfo").style.display = "block";
                document.getElementById("preSaleData").style.display = "block";

                // Initialize Contract Instance
                contract = new web3.eth.Contract(contractABI, contractAddress);

                // Fetch Pre-Sale Data
                updatePreSaleData();
                setInterval(updatePreSaleData, 10000); // Auto-refresh every 10 seconds
            } catch (error) {
                console.error("Wallet connection rejected", error);
            }
        } else {
            alert("Please install MetaMask to connect.");
        }
    });

    async function updatePreSaleData() {
        if (!contract) return;
        
        document.getElementById("loadingMessage").style.display = "block";

        try {
            let startTime = await contract.methods.startTime().call();
            let formattedStartTime = new Date(startTime * 1000).toLocaleString();
            let totalRaisedWei = await contract.methods.totalRaised().call();
            let totalRaisedEth = web3.utils.fromWei(totalRaisedWei, "ether");
            let remainingTokens = await contract.methods.totalTokensForSale().call();

            document.getElementById("startTime").textContent = formattedStartTime;
            document.getElementById("totalRaised").textContent = totalRaisedEth;
            document.getElementById("remainingTokens").textContent = remainingTokens;

        } catch (error) {
            console.error("Error fetching contract data", error);
        }

        document.getElementById("loadingMessage").style.display = "none";
    }

    // Buy Tokens Function
    document.getElementById("buyButton").addEventListener("click", async () => {
        const amount = document.getElementById("amount").value;
        if (amount <= 0) {
            alert("Please enter a valid amount.");
            return;
        }

        if (!contract) {
            alert("Smart contract not connected.");
            return;
        }

        try {
            const accounts = await web3.eth.getAccounts();
            const account = accounts[0];

            // Fetch Token Price from Contract
            const tokenPriceWei = await contract.methods.rate().call();
            const totalCostWei = web3.utils.toBN(tokenPriceWei).mul(web3.utils.toBN(amount));

            // Send Transaction
            document.getElementById("statusMessage").textContent = "Processing transaction...";
            await contract.methods.buyTokens(amount).send({
                from: account,
                value: totalCostWei
            });

            document.getElementById("statusMessage").textContent = "Purchase successful!";
            updatePreSaleData(); // Refresh Data After Purchase
        } catch (error) {
            console.error("Transaction failed", error);
            document.getElementById("statusMessage").textContent = "Transaction failed.";
        }
    });

</script>
</body>
</html>
