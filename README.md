<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Token Presale</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 20px;
            background-color: #f4f4f9;
            color: #333;
        }
        h1 {
            color: #4CAF50;
        }
        form {
            background: #fff;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
            max-width: 400px;
            margin: 20px auto;
        }
        label {
            display: block;
            margin-bottom: 8px;
            font-weight: bold;
        }
        input[type="number"] {
            width: 100%;
            padding: 10px;
            margin-bottom: 20px;
            border: 1px solid #ccc;
            border-radius: 4px;
            font-size: 16px;
        }
        button {
            background-color: #4CAF50;
            color: white;
            padding: 10px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
        }
        button:hover {
            background-color: #45a049;
        }
        #connectButton {
            margin-bottom: 20px;
        }
        .message {
            margin-top: 20px;
            padding: 10px;
            border-radius: 4px;
            text-align: center;
        }
        .success {
            background-color: #d4edda;
            color: #155724;
        }
        .error {
            background-color: #f8d7da;
            color: #721c24;
        }
    </style>
</head>
<body>
    <h1>Token Presale</h1>
    <p>Welcome to our token presale! Connect your MetaMask wallet and participate.</p>
    <button id="connectButton">Connect MetaMask</button>
    <form id="buyTokensForm">
        <label for="ethAmount">ETH Amount (Min: 0.003 ETH, Max: 0.15 ETH):</label>
        <input type="number" id="ethAmount" step="0.001" min="0.003" max="0.15" required>
        <button type="submit">Buy Tokens</button>
    </form>
    <div id="message" class="message"></div>

    <script src="https://cdn.jsdelivr.net/npm/ethers@5.7.2/dist/ethers.umd.min.js"></script>
    <script>
        
        const presaleContractABI =[{"inputs":[],"stateMutability":"nonpayable","type":"constructor"},{"anonymous":false,"inputs":[{"indexed":true,"internalType":"address","name":"owner","type":"address"},{"indexed":true,"internalType":"address","name":"spender","type":"address"},{"indexed":false,"internalType":"uint256","name":"value","type":"uint256"}],"name":"Approval","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"internalType":"address","name":"from","type":"address"},{"indexed":true,"internalType":"address","name":"to","type":"address"},{"indexed":false,"internalType":"uint256","name":"value","type":"uint256"}],"name":"Transfer","type":"event"},{"inputs":[{"internalType":"address","name":"","type":"address"},{"internalType":"address","name":"","type":"address"}],"name":"allowance","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"address","name":"spender","type":"address"},{"internalType":"uint256","name":"value","type":"uint256"}],"name":"approve","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"nonpayable","type":"function"},{"inputs":[{"internalType":"address","name":"owner","type":"address"}],"name":"balanceOf","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"balances","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"decimals","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"name","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"symbol","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"totalSupply","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"address","name":"to","type":"address"},{"internalType":"uint256","name":"value","type":"uint256"}],"name":"transfer","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"nonpayable","type":"function"},{"inputs":[{"internalType":"address","name":"from","type":"address"},{"internalType":"address","name":"to","type":"address"},{"internalType":"uint256","name":"value","type":"uint256"}],"name":"transferFrom","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"nonpayable","type":"function"}]  ;

        // Sözleşme Adresi
        const presaleContractAddress = "0x951AfD9E496030Fb24a55c55C5a2f2f1520Bdfa5";

        // MetaMask Bağlantısı
        const connectButton = document.getElementById('connectButton');
        const buyTokensForm = document.getElementById('buyTokensForm');
        const messageDiv = document.getElementById('message');

        let userAddress = null;

        // MetaMask Bağlantısını Yönet
        connectButton.addEventListener('click', async () => {
            if (typeof window.ethereum !== 'undefined') {
                try {
                    const accounts = await ethereum.request({ method: 'eth_requestAccounts' });
                    userAddress = accounts[0];
                    showMessage(`Connected: ${userAddress}`, 'success');
                } catch (error) {
                    showMessage(`Error connecting to MetaMask: ${error.message}`, 'error');
                }
            } else {
                showMessage('MetaMask is not installed!', 'error');
            }
        });

        // Token Satın Alma İşlemi
        buyTokensForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            if (!userAddress) {
                showMessage('Please connect your MetaMask wallet first.', 'error');
                return;
            }

            const ethAmount = document.getElementById('ethAmount').value;
            const minETH = 0.003;
            const maxETH = 0.15;

            if (ethAmount < minETH || ethAmount > maxETH) {
                showMessage(`ETH amount must be between ${minETH} and ${maxETH}`, 'error');
                return;
            }

            const weiAmount = ethers.utils.parseEther(ethAmount);

            if (typeof window.ethereum !== 'undefined') {
                const provider = new ethers.providers.Web3Provider(window.ethereum);
                const signer = provider.getSigner();
                const presaleContract = new ethers.Contract(presaleContractAddress, presaleContractABI, signer);

                try {
                    const tx = await presaleContract.buyTokens({ value: weiAmount });
                    await tx.wait();
                    showMessage('Tokens purchased successfully!', 'success');
                } catch (error) {
                    showMessage(`Error purchasing tokens: ${error.message}`, 'error');
                }
            } else {
                showMessage('MetaMask is not installed!', 'error');
            }
        });

        // Mesaj Gösterme Fonksiyonu
        function showMessage(message, type) {
            messageDiv.textContent = message;
            messageDiv.className = `message ${type}`;
        }
    </script>
</body>
</html>
