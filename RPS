// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract RPS_LizardSpock {
    enum Move { None, Rock, Paper, Scissors, Lizard, Spock }

    struct Player {
        address addr;
        bytes32 commitHash;
        Move move;
        uint256 betAmount;
        bool revealed;
    }

    Player public player1;
    Player public player2;
    uint256 public commitDeadline;
    uint256 public revealDeadline;

    address[] private allowedPlayers = [
        0x5B38Da6a701c568545dCfcB03FcB875f56beddC4,
        0xAb8483F64d9C6d1EcF9b849Ae677dD3315835cb2,
        0x4B20993Bc481177ec7E8f571ceCaE8A9e22C02db,
        0x78731D3Ca6b7E34aC0F824c42a7cC18A495cabaB
    ];

    modifier onlyAllowedPlayers() {
        bool isAllowed = false;
        for (uint i = 0; i < allowedPlayers.length; i++) {
            if (msg.sender == allowedPlayers[i]) {
                isAllowed = true;
                break;
            }
        }
        require(isAllowed, "You are not allowed to play this game.");
        _;
    }

    event GameStarted(address player1, address player2);
    event CommitReceived(address player);
    event RevealReceived(address player, Move move);
    event GameResult(address winner, uint256 amount);
    event GameTied();

    constructor() {
        commitDeadline = block.timestamp + 5 minutes;
        revealDeadline = commitDeadline + 5 minutes;
    }

    function joinGame(bytes32 _commitHash) external payable onlyAllowedPlayers {
        require(msg.value > 0, "Must place a bet");
        
        if (player1.addr == address(0)) {
            player1 = Player(msg.sender, _commitHash, Move.None, msg.value, false);
        } else {
            require(player2.addr == address(0), "Game already full");
            player2 = Player(msg.sender, _commitHash, Move.None, msg.value, false);
            emit GameStarted(player1.addr, player2.addr);
        }

        emit CommitReceived(msg.sender);
    }

    function revealMove(string memory _secret, Move _move) external {
        require(block.timestamp > commitDeadline && block.timestamp <= revealDeadline, "Not in reveal phase");
        require(_move >= Move.Rock && _move <= Move.Spock, "Invalid move");

        bytes32 computedHash = keccak256(abi.encodePacked(_secret, _move));

        if (msg.sender == player1.addr) {
            require(player1.commitHash == computedHash, "Invalid commitment");
            player1.move = _move;
            player1.revealed = true;
        } else if (msg.sender == player2.addr) {
            require(player2.commitHash == computedHash, "Invalid commitment");
            player2.move = _move;
            player2.revealed = true;
        } else {
            revert("Not a player");
        }

        emit RevealReceived(msg.sender, _move);

        if (player1.revealed && player2.revealed) {
            _determineWinner();
        }
    }

    function _determineWinner() internal {
        require(player1.revealed && player2.revealed, "Both players must reveal");

        Move p1Move = player1.move;
        Move p2Move = player2.move;

        address winner;
        uint256 totalBet = player1.betAmount + player2.betAmount;

        if (p1Move == p2Move) {
            payable(player1.addr).transfer(player1.betAmount);
            payable(player2.addr).transfer(player2.betAmount);
            emit GameTied();
        } else if (_winsAgainst(p1Move, p2Move)) {
            winner = player1.addr;
            payable(player1.addr).transfer(totalBet);
        } else {
            winner = player2.addr;
            payable(player2.addr).transfer(totalBet);
        }

        emit GameResult(winner, totalBet);
        _resetGame();
    }

    function _winsAgainst(Move _a, Move _b) internal pure returns (bool) {
        return (_a == Move.Rock && (_b == Move.Scissors || _b == Move.Lizard)) ||
               (_a == Move.Paper && (_b == Move.Rock || _b == Move.Spock)) ||
               (_a == Move.Scissors && (_b == Move.Paper || _b == Move.Lizard)) ||
               (_a == Move.Lizard && (_b == Move.Spock || _b == Move.Paper)) ||
               (_a == Move.Spock && (_b == Move.Scissors || _b == Move.Rock));
    }

    function _resetGame() internal {
        delete player1;
        delete player2;
        commitDeadline = block.timestamp + 5 minutes;
        revealDeadline = commitDeadline + 5 minutes;
    }

    function claimTimeout() external {
        require(block.timestamp > revealDeadline, "Reveal phase not ended");

        if (!player1.revealed && player2.revealed) {
            payable(player2.addr).transfer(address(this).balance);
            emit GameResult(player2.addr, address(this).balance);
        } else if (!player2.revealed && player1.revealed) {
            payable(player1.addr).transfer(address(this).balance);
            emit GameResult(player1.addr, address(this).balance);
        } else {
            payable(player1.addr).transfer(player1.betAmount);
            payable(player2.addr).transfer(player2.betAmount);
            emit GameTied();
        }
        _resetGame();
    }
}
