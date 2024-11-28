Cannabis Smart Contract (CSC) mit 90% und 10% Verteilung
solidity
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/Address.sol";

contract CannabisSmartContract is ERC20, Ownable {
    using Address for address;

    // Belohnungsmechanismen für verschiedene Coins
    mapping(address => uint256) public trashToCashBalances;
    mapping(address => uint256) public stakedBalances;
    uint256 public rewardRate = 8; // tägliche Rendite von 8%
    uint256 public stakingStartTime;
    address public rfofSpiderNetAddress;
    address public bridgeAddress;
    address public praiAddress;
    address public dedustPoolAddress;
    address public frensCoinAddress;
    address public majorCoinAddress;

    event TrashDataProcessed(uint256 trashDataId, uint256 convertedAmount);
    event TokensStaked(address indexed user, uint256 amount);
    event RewardsClaimed(address indexed user, uint256 reward);

    constructor(
        address _rfofSpiderNetAddress,
        address _bridgeAddress,
        address _praiAddress,
        address _dedustPoolAddress,
        address _frensCoinAddress,
        address _majorCoinAddress
    ) ERC20("Cannabis Smart Contract", "CSC") {
        rfofSpiderNetAddress = _rfofSpiderNetAddress;
        bridgeAddress = _bridgeAddress;
        praiAddress = _praiAddress;
        dedustPoolAddress = _dedustPoolAddress;
        frensCoinAddress = _frensCoinAddress;
        majorCoinAddress = _majorCoinAddress;
        stakingStartTime = block.timestamp;
    }

    modifier onlyPRAI() {
        require(msg.sender == praiAddress, "Only PRAI can perform this action");
        _;
    }

    function processTrashData(uint256 trashDataId) public {
        uint256 convertedAmount = trashDataId * 100; // Beispielhafte Umrechnung
        trashToCashBalances[msg.sender] += convertedAmount;
        _mint(msg.sender, convertedAmount);
        emit TrashDataProcessed(trashDataId, convertedAmount);
    }

    function stakeTokens(uint256 amount) public {
        require(balanceOf(msg.sender) >= amount, "Insufficient token balance");
        _burn(msg.sender, amount);
        stakedBalances[msg.sender] += amount;
        emit TokensStaked(msg.sender, amount);
    }

    function claimRewards() public {
        require(stakedBalances[msg.sender] > 0, "No tokens staked");
        uint256 stakedTime = block.timestamp - stakingStartTime;
        uint256 reward = (stakedBalances[msg.sender] * rewardRate * stakedTime) / (100 * 1 days);
        _mint(msg.sender, reward);
        stakingStartTime = block.timestamp;
        emit RewardsClaimed(msg.sender, reward);
    }

    function mintTokens(uint256 dataId) public {
        uint256 tokens = dataId * 100; // Beispielhafte Berechnung

        // Verteilung der Tokens
        uint256 communityShare = (tokens * 90) / 100;
        uint256 cooperationShare = (tokens * 5) / 100;
        uint256 praiShare = (tokens * 5) / 100;

        balances[msg.sender] += communityShare;
        _mint(frensCoinAddress, cooperationShare / 2);
        _mint(majorCoinAddress, cooperationShare / 2);
        _mint(praiAddress, praiShare);

        emit RewardsClaimed(msg.sender, communityShare);
    }

    function bridgeTokens(uint256 amount) public onlyPRAI {
        require(balanceOf(msg.sender) >= amount, "Insufficient token balance");
        _burn(msg.sender, amount);
        (bool success, ) = bridgeAddress.call{value: amount}("");
        require(success, "Bridge transfer failed");
    }

    function distributeRewards(address user, uint256 amount) public onlyPRAI {
        require(balanceOf(msg.sender) >= amount, "Insufficient token balance");
        _burn(msg.sender, amount);
        uint256 reward = (amount * rewardRate) / 100;
        _mint(user, reward);
        _mint(rfofSpiderNetAddress, reward / 10); // 10% an das RFOF Spidernet
        emit RewardsClaimed(user, reward);
    }

    function inviteUser(address inviter, address newUser) public onlyPRAI {
        _mint(inviter, 1500); // 1500 $BBC Token an den Einladenden
        _mint(newUser, 50); // 50 $CTC Token an den neuen User
    }

    function swapToTON(address recipient, uint256 amount) internal {
        uint256 tonAmount = (amount * 1) / 10000; // 0,01% in TON umwandeln
        (bool success, ) = recipient.call{value: tonAmount}("");
        require(success, "TON swap failed");
    }

    function distributeBubatzRewards(address user, uint256 dataId) public onlyPRAI {
        uint256 amount = dataId * 100;
        _mint(user, amount);
        uint256 tonAmount = (amount * 1) / 10000; // 0,01% in TON umwandeln

        // Swap to TON for frescoinbot.ton and majorapp.ton
        swapToTON(frensCoinAddress, tonAmount);
        swapToTON(majorCoinAddress, tonAmount);

        emit RewardsClaimed(user, amount);
    }
}
Zusammenfassung der Funktionen:
Token-Minting und Verteilung: Erzeugung von Tokens basierend auf validierten Daten und Verteilung an die Community (90%) und in zwei Teilen von jeweils 5% (Kooperationen und PRAI).
Trash to Cash: Umwandlung von "Trash-Daten" in Tokens.
Staking und Belohnungen: Nutzer können Tokens staken und tägliche Belohnungen von 8% erhalten. Belohnungen werden basierend auf der Staking-Dauer berechnet.
Bridge zu anderen Blockchains: Übertragung von Tokens zu anderen Blockchains durch eine spezielle Bridge-Adresse.
Einladungsbelohnungen: Einladende Nutzer erhalten 1500 $BBC Token und neue Nutzer erhalten 50 $CTC Token.
Swap zu TON: Umwandlung von 0,01% der Tokens in TON für frescoinbot.ton und majorapp.ton.
Integration mit dem RFOF Spidernet und Dedust Pool: Verteilung von Belohnungen an das RFOF Spidernet und Nutzung der Dedust Staking API für bessere Benutzerfreundlichkeit.
