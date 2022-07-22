# UniswapV2Factory源码解析

## 代码

```
pragma solidity =0.5.16;

import './interfaces/IUniswapV2Factory.sol';
import './UniswapV2Pair.sol';

contract UniswapV2Factory is IUniswapV2Factory {
    address public feeTo;
    address public feeToSetter;

    mapping(address => mapping(address => address)) public getPair;
    address[] public allPairs;

    event PairCreated(address indexed token0, address indexed token1, address pair, uint);

    constructor(address _feeToSetter) public {
        feeToSetter = _feeToSetter;
    }

    function allPairsLength() external view returns (uint) {
        return allPairs.length;
    }

    function createPair(address tokenA, address tokenB) external returns (address pair) {
        //需要两个代币是不同代币
        require(tokenA != tokenB, 'UniswapV2: IDENTICAL_ADDRESSES');
        //token0地址较小代币，token1地址较大代币
        (address token0, address token1) = tokenA < tokenB ? (tokenA, tokenB) : (tokenB, tokenA);
        //代币地址不为零
        require(token0 != address(0), 'UniswapV2: ZERO_ADDRESS');
        //代币合约不存在
        require(getPair[token0][token1] == address(0), 'UniswapV2: PAIR_EXISTS'); // single check is sufficient
        //获得创建合约代码
        bytes memory bytecode = type(UniswapV2Pair).creationCode;
        //加盐
        bytes32 salt = keccak256(abi.encodePacked(token0, token1));
        //获得合约地址
        assembly {
            pair := create2(0, add(bytecode, 32), mload(bytecode), salt)
        }
        //初始化合约
        IUniswapV2Pair(pair).initialize(token0, token1);
        getPair[token0][token1] = pair;
        getPair[token1][token0] = pair; // populate mapping in the reverse direction
        allPairs.push(pair);
        //发布合约
        emit PairCreated(token0, token1, pair, allPairs.length);
    }
    //设置授权地址
    function setFeeTo(address _feeTo) external {
        require(msg.sender == feeToSetter, 'UniswapV2: FORBIDDEN');
        feeTo = _feeTo;
    }
    //更改授权人
    function setFeeToSetter(address _feeToSetter) external {
        require(msg.sender == feeToSetter, 'UniswapV2: FORBIDDEN');
        feeToSetter = _feeToSetter;
    }
}

```

### address public feeTo

收税地址

### address public feeToSetter

设置收税地址的权限

### 交易对的合约地址
mapping(address => mapping(address => address)) public getPair


