### **Lab 9 : Intégration Complète de l’Instant Payment Hub (Backend & Frontend)**

#### **Description du Lab**

Dans ce lab, vous allez intégrer le contrat **Instant Payment Hub** que vous avez développé dans les précédents labs avec un **frontend** complet. Vous allez utiliser **React.js** (ou autre technologie front-end) pour créer une interface utilisateur qui permettra de :

*   Se connecter à **Metamask**.
*   Déposer des fonds dans le contrat.
*   Effectuer des paiements instantanés.
*   Retirer des fonds du contrat.
*   Visualiser les transactions et le solde d'un utilisateur.

Le **backend** du projet repose sur **Hardhat** et **solidity**, tandis que le **frontend** sera construit avec **React.js** pour l'interaction avec le contrat déployé.

* * *

### **Prérequis**

Avant de commencer ce lab, assurez-vous d'avoir configuré votre environnement comme suit :

*   **Node.js >= 16.x**
*   **Metamask** ou un autre wallet Ethereum installé dans votre navigateur
*   **Compte Ethereum sur Sepolia Testnet**
*   **GitHub Codespaces** ou un IDE local avec **Hardhat**, **solidity**, **React.js**, et **Ethers.js** installés
*   **Infura/Alchemy** pour interagir avec Sepolia Testnet

* * *

### **Objectifs du Lab**

1.  Intégrer un contrat **Instant Payment Hub** déployé sur **Sepolia** avec un **frontend** en **React.js**.
2.  Créer des fonctionnalités permettant d'interagir avec le contrat via Metamask, comme le dépôt de fonds, l'exécution de paiements, et le retrait de fonds.
3.  Implémenter une interface utilisateur fonctionnelle avec **React.js** pour que l'utilisateur puisse voir son solde et effectuer des transactions.
4.  Déployer l'application front-end pour tester son intégration avec le backend.

* * *

### **Étapes du Lab**

#### **1. Préparer l'environnement**  
 

**Installer les dépendances** en utilisant npm :  
  
```bash  
cd lab-integration-instant-payment-hub

npm install

```

**Vérifier la configuration de Hardhat** pour vous assurer que vous avez un contrat déployé sur Sepolia, comme mentionné dans le **Lab 3**. Si vous avez déjà déployé le contrat, récupérez son adresse.  
 

* * *

#### **2. Créer le Backend avec Hardhat**

Si vous n'avez pas encore créé le contrat, voici un rappel de l'implémentation d'un **Instant Payment Hub** sécurisé dans contracts/InstantPaymentHub.sol :

```solidity

// SPDX-License-Identifier: MIT

pragma solidity ^0.8.20;

import "@openzeppelin/contracts/access/Ownable.sol";

import "@openzeppelin/contracts/security/Pausable.sol";

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

import "@openzeppelin/contracts/utils/math/SafeMath.sol";

contract InstantPaymentHub is Ownable, Pausable, ReentrancyGuard {

    using SafeMath for uint256;

    mapping(address => uint256) public balances;

    event PaymentMade(address indexed sender, address indexed receiver, uint256 amount);

    modifier whenNotPaused() {

        require(!paused(), "Contract is paused");

        _;

    }

    constructor() {

        // Propriétaire défini à l'adresse qui déploie le contrat

    }

    // Fonction pour déposer de l'ether dans le contrat

    function deposit() public payable whenNotPaused {

        balances[msg.sender] = balances[msg.sender].add(msg.value);

    }

    // Fonction pour effectuer un paiement instantané entre utilisateurs

    function instantPayment(address recipient, uint256 amount) public whenNotPaused nonReentrant {

        require(balances[msg.sender] >= amount, "Insufficient balance");

        balances[msg.sender] = balances[msg.sender].sub(amount);

        balances[recipient] = balances[recipient].add(amount);

        emit PaymentMade(msg.sender, recipient, amount);

    }

    // Fonction pour retirer des fonds du contrat

    function withdraw(uint256 amount) public whenNotPaused nonReentrant {

        require(balances[msg.sender] >= amount, "Insufficient balance");

        payable(msg.sender).transfer(amount);

        balances[msg.sender] = balances[msg.sender].sub(amount);

    }

    // Fonction pour mettre en pause le contrat (uniquement accessible par le propriétaire)

    function pause() public onlyOwner {

        _pause();

    }

    // Fonction pour reprendre les opérations du contrat

    function unpause() public onlyOwner {

        _unpause();

    }

}

```

Déployez ce contrat en utilisant **Hardhat**, comme vous l'avez vu dans le **Lab 3**. N'oubliez pas de récupérer l'adresse du contrat déployé sur Sepolia.

* * *

#### **3. Créer le Frontend avec React.js**

1.  **Initialiser un projet React.js** :  
     

Si vous n'avez pas encore créé un projet React.js, utilisez la commande suivante :  
  
```bash  
npx create-react-app instant-payment-hub-frontend

cd instant-payment-hub-frontend

npm install ethers

```

**Créer les fichiers nécessaires** pour interagir avec le contrat :

*   **src/App.js** : Ce fichier contiendra le code pour interagir avec le contrat via **Ethers.js** et afficher les informations de l'utilisateur.

Voici un exemple de code pour App.js :

```javascript

import React, { useState, useEffect } from "react";

import { ethers } from "ethers";

import "./App.css";

function App() {

  const [account, setAccount] = useState(null);

  const [contract, setContract] = useState(null);

  const [balance, setBalance] = useState(0);

  const [depositAmount, setDepositAmount] = useState("");

  const [paymentAmount, setPaymentAmount] = useState("");

  const [recipient, setRecipient] = useState("");

  // Connexion avec Metamask

  const connectWallet = async () => {

    if (window.ethereum) {

      const provider = new ethers.providers.Web3Provider(window.ethereum);

      await provider.send("eth_requestAccounts", []);

      const signer = provider.getSigner();

      setAccount(await signer.getAddress());

      const contractAddress = "VOTRE_ADRESSE_DE_CONTRAT"; // Remplacez par l'adresse du contrat déployé

      const contractABI = [

        "function deposit() public payable",

        "function instantPayment(address recipient, uint256 amount) public",

        "function withdraw(uint256 amount) public",

        "function balances(address) public view returns (uint256)"

      ];

      const contract = new ethers.Contract(contractAddress, contractABI, signer);

      setContract(contract);

      const userBalance = await contract.balances(account);

      setBalance(ethers.utils.formatEther(userBalance));

    } else {

      alert("Please install Metamask!");

    }

  };

  // Déposer des fonds

  const depositFunds = async () => {

    if (contract && depositAmount > 0) {

      const tx = await contract.deposit({ value: ethers.utils.parseEther(depositAmount) });

      await tx.wait();

      alert("Deposit successful");

      loadBalance();

    }

  };

  // Effectuer un paiement

  const makePayment = async () => {

    if (contract && paymentAmount > 0 && recipient) {

      const tx = await contract.instantPayment(recipient, ethers.utils.parseEther(paymentAmount));

      await tx.wait();

      alert("Payment successful");

      loadBalance();

    }

  };

  // Retirer des fonds

  const withdrawFunds = async () => {

    if (contract && balance > 0) {

      const tx = await contract.withdraw(ethers.utils.parseEther(balance));

      await tx.wait();

      alert("Withdrawal successful");

      loadBalance();

    }

  };

  // Charger le solde

  const loadBalance = async () => {

    const userBalance = await contract.balances(account);

    setBalance(ethers.utils.formatEther(userBalance));

  };

  useEffect(() => {

    if (window.ethereum) {

      connectWallet();

    }

  }, []);

  return (

    <div className="App">

      <h1>Instant Payment Hub</h1>

      {account ? (

        <div>

          <p>Connected account: {account}</p>

          <p>Balance: {balance} ETH</p>

          <div>

            <h2>Deposit Funds</h2>

            <input

              type="number"

              placeholder="Amount to deposit"

              value={depositAmount}

              onChange={(e) => setDepositAmount(e.target.value)}

            />

            <button onClick={depositFunds}>Deposit</button>

          </div>

          <div>

            <h2>Make Payment</h2>

            <input

              type="text"

              placeholder="Recipient address"

              value={recipient}

              onChange={(e) => setRecipient(e.target.value)}

            />

            <input

              type="number"

              placeholder="Amount to pay"

              value={paymentAmount}

              onChange={(e) => setPaymentAmount(e.target.value)}

            />

            <button onClick={makePayment}>Pay</button>

          </div>

          <div>

            <h2>Withdraw Funds</h2>

            <button onClick={withdrawFunds}>Withdraw</button>

          </div>

        </div>

      ) : (

        <button onClick={connectWallet}>Connect Metamask</button>

      )}

    </div>

  );

}

export default App;

```

**Configurer les composants React** pour interagir avec le contrat, comme le dépôt de fonds, le paiement et le retrait.  
 

* * *

#### **4. Tester l'application React.js**

**Lancer le projet React** :  
  
```bash  
npm start

```

**Ouvrir l'application dans le navigateur** et interagir avec le contrat **Instant Payment Hub** en utilisant **Metamask** pour :  
 

*   Déposer des fonds.
*   Effectuer des paiements instantanés.
*   Retirer des fonds.

* * *

#### **5. Vérification des transactions sur Etherscan**

*   Une fois que vous avez effectué des transactions, vous pouvez vérifier leur état sur **Sepolia Etherscan**. Recherchez l'adresse du contrat déployé et consultez les transactions de paiement.

* * *

### **Ressources supplémentaires**

*   **solidity Documentation** : solidity Documentation
*   **Hardhat Documentation** : Hardhat Documentation
*   **Ethers.js Documentation** : Ethers.js Documentation
*   **React Documentation** : React Docs
*   **Metamask Documentation** : [Metamask Docs](https://metamask.io/)  
     

* * *

### **Fichiers du Projet**

Voici un aperçu des fichiers et dossiers dans le repo pour ce lab :

```pgsql

lab-integration-instant-payment-hub/

│

├── frontend/

│   ├── src/

│   │   ├── App.js

│   │   └── index.js

│   ├── public/

│   │   └── index.html

│   └── package.json

│

├── contracts/

│   └── InstantPaymentHub.sol

│

├── scripts/

│   ├── deploy.js

│   └── interact.js

│

├── hardhat.config.js

├── package.json

└── README.md

```

* * *

### **Objectifs de l'étudiant après ce lab**

*   Intégrer un contrat Ethereum avec un **frontend React.js**.
*   Créer une interface utilisateur permettant des actions comme le **dépôt**, **paiement** et **retrait**.
*   Tester et déployer l'application front-end sur le testnet **Sepolia**.


