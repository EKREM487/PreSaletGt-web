<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ethereum Akıllı Sözleşme</title>

    <!-- Nonce ile harici kaynağa izin veriyoruz -->
    <meta http-equiv="Content-Security-Policy" content="
        default-src 'self'; 
        script-src 'self' https://cdn.jsdelivr.net 'nonce-r4nd0mN0nce';
        connect-src 'self' https://api.etherscan.io;
    ">

    <!-- Web3.js dosyasını nonce ile çağırıyoruz -->
    <script nonce="r4nd0mN0nce" src="https://cdn.jsdelivr.net/npm/web3@latest/dist/web3.min.js"></script>

    <style>
        body { font-family: Arial, sans-serif; text-align: center; margin: 20px; }
        #accountInfo, #contractInfo { margin-top: 20px; display: none; }
    </style>
</head>
<body>

    <h1>Ethereum Akıllı Sözleşme Entegrasyonu</h1>
    <button id="connectWallet">Cüzdanı Bağla</button>

    <div id="accountInfo">
        <h3>Bağlı Hesap:</h3>
        <p><strong>Hesap:</strong> <span id="accountAddress">Bağlı değil</span></p>
        <p><strong>ETH Bakiyesi:</strong> <span id="ethBalance">0</span> ETH</p>
    </div>

    <div id="contractInfo">
        <h3>Akıllı Sözleşme Bilgileri</h3>
        <p><strong>Toplam ETH:</strong> <span id="totalETH">Yükleniyor...</span></p>
        <button id="buyTokens">tGt Token Satın Al</button>
        <p id="statusMessage"></p>
    </div>

    <!-- Nonce ile güvenli inline script -->
    <script nonce="r4nd0mN0nce">
        let web3;
        let contract;
        let userAccount;

        const contractAddress = "0x298Af6b1ac1c846a5E2eB541A4955c4994f3D882";

        async function fetchABI() {
            const url = `https://api-sepolia.etherscan.io/api?module=contract&action=getabi&address=0x298af6b1ac1c846a5e2eb541a4955c4994f3d882}`;
            
            try {
                const response = await fetch(url);
                const data = await response.json();
                
                if (data.status === "1") {
                    return JSON.parse(data.result);
                } else {
                    console.error("Etherscan API hatası:", data.message);
                }
            } catch (error) {
                console.error("ABI çekme hatası:", error);
            }
        }

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

                    const abi = await fetchABI();
                    contract = new web3.eth.Contract(abi, contractAddress);

                    updateContractInfo();
                } catch (error) {
                    console.error("Cüzdan bağlantı hatası:", error);
                    alert("Cüzdan bağlanamadı.");
                }
            } else {
                alert("Lütfen MetaMask veya Web3 uyumlu bir cüzdan yükleyin.");
            }
        }

        async function updateContractInfo() {
            try {
                const totalRaised = await contract.methods.totalRaised().call();
                document.getElementById('totalETH').textContent = web3.utils.fromWei(totalRaised, 'ether') + " ETH";
            } catch (error) {
                console.error("Akıllı sözleşme verileri alınamadı:", error);
            }
        }

        async function buyTokens() {
            try {
                const amount = prompt("Kaç tGt almak istiyorsunuz?");
                if (amount <= 0) {
                    alert("Lütfen geçerli bir miktar girin.");
                    return;
                }

                const tokenPrice = web3.utils.toWei('0.01', 'ether'); 
                const totalCost = web3.utils.toBN(tokenPrice).mul(web3.utils.toBN(amount));

                document.getElementById('statusMessage').textContent = "İşlem gerçekleştiriliyor...";

                await contract.methods.buyTokens().send({
                    from: userAccount,
                    value: totalCost
                });

                document.getElementById('statusMessage').textContent = "Satın alma başarılı!";
                updateContractInfo();
            } catch (error) {
                console.error("Satın alma hatası:", error);
                document.getElementById('statusMessage').textContent = "İşlem başarısız.";
            }
        }

        document.getElementById('connectWallet').addEventListener('click', connectWallet);
        document.getElementById('buyTokens').addEventListener('click', buyTokens);
    </script>

</body>
</html>

