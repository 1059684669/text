# smart contracts：structure

~~~solidity
// SPDX-License-Identifier: MIT
#### pragma solidity ^0.8.7;
// pragma solidity ^0.8.0;
// pragma solidity >=0.8.0 <0.9.0;
import“./h.sol”；
contract  名字{
	uint a；
struct  Name{
}
Name[] public name;
  function name(uint number,string memory _name) public {     //相当于函数
	}
}
~~~



### msg:全局变量

msg.sender:发起当前交易的账户地址,通常用于验证调用者身份或在合约中记录谁执行了某个操作。

msg.value:随当前交易发送的以太币数量，单位是 wei（以太坊的最小单位）。例如在合约中接收以太币

msg.data:当前交易的完整数据负载

msg.gas:当前交易可用的气体总量

msg.sig:当前交易的前四个字节，表示调用的函数选择器. 用于确定调用的是哪个函数。



在 Solidity 中 constant、view、pure 三个函数修饰词的作用是告诉编译器，函数不改变 / 不读取状态变量，这样函数执行就可以不消耗 gas 了（是**完全不消耗**！），因为不需要矿工来验证，所以用好这几个关键词对省 gas 很重要。

- constant: 常量，不可更改
- view: 可读取但不可更改合约中的状态变量
- pure: 不可读取且不可更改合约中的状态变量



## block：

block.number:当前区块的编号。

block.timestamp:当前的时间戳。

block.difficulty:当前的难度。

block.gaslimit:当前区块的gas限制。

block.coinbase:当前区块的矿工地址。

~~~solidity
function getBlockInfo() public view returns (uint, uint, uint, address) {
  return (
    block.number,    // 当前区块的编号
    block.timestamp,  // 当前区块的时间戳
    block.difficulty,  // 当前区块的难度
    block.coinbase   // 当前区块的矿工地址
   );
}
~~~

constant（常量），immutable（不可变量）

链api调用：通过chainlink

外部函数需要用 this.f  调用

selector：可以返回ABI函数选择器：this.f.selector

bytes.concat：可以连接任意数量的bytes值，该函数返回一个单一的bytes memory数组

#### 分配内存数组：

可以使用 `new` 操作符创建动态长度的内存数组。 与存储数组不同，内存数组 **不能** 调整大小

uint[] memory a=new uint[](7);

#### 数组字面量：

**uint[2][4]:即为[1,2],若要转换为相同的，则需[uint(1),2];**

`[1, 2, 3]` 的类型是 `uint8[3] memory`，因为这些常量的类型都是 `uint8`。 如果你希望结果为 `uint[3] memory` 类型，你需要将第一个元素转换为 `uint`。

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.16 <0.9.0;

contract C {
    function f() public pure {
        g([uint(1), 2, 3]);
    }
    function g(uint[3] memory) public pure {
        // ...
    }
}
```



如果你想初始化动态大小的数组，你必须逐个赋值

数组功能：length，push()，push(x)，pop()



#### 结构体：

```solidity
struct Funder {
    address addr;
    uint amount;
}
Funder[] public funder;   //结构体的数组定义
```

#### 映射：即通过一个字段查找该字段的其他信息，如地址和ID对应

```solidity
contract MappingExampleWithNames {
    mapping(address user => uint balance) public balances;

    function update(uint newBalance) public {
        balances[msg.sender] = newBalance;
    }
}
```





## 引用类型：

如果使用引用类型，必须明确提供存储该类型的数据区域：

`memory` （其生命周期仅限于外部函数调用）

`storage` （存储状态变量的地点，其生命周期限于合约的生命周期）

`calldata` （包含函数参数的特殊数据位置，是一个不可修改的、非持久的区域，用于存储函数参数，其行为大致类似于内存）



#### 以太单位：

子单位后缀的唯一效果是乘以十的幂

```solidity
assert(1 wei == 1);
assert(1 gwei == 1e9);
assert(1 ether == 1e18);
```

```solidity
//如果你想以天为单位解释函数参数
function f(uint start, uint daysAfter) public {
    if (block.timestamp >= start + daysAfter * 1 days) {
        // ...
    }
}
```



#### ABI 编码和解码函数

abi.decode(bytes memory encodedData, (...)) returns (...)：对给定数据进行 ABI 解码，类型在括号中作为第二个参数给出。示例：(uint a, uint[2] memory b, bytes memory c) = abi.decode(data, (uint, uint[2], bytes))

abi.encode(...) returns (bytes memory)：对给定参数进行 ABI 编码

abi.encodePacked(...) returns (bytes memory)：对给定参数执行 packed encoding。请注意，打包编码可能会产生歧义！

abi.encodeWithSelector(bytes4 selector, ...) returns (bytes memory)：对给定参数进行 ABI 编码，从第二个参数开始，并在前面添加给定的四字节选择器

abi.encodeWithSignature(string memory signature, ...) returns (bytes memory)：等同于 abi.encodeWithSelector(bytes4(keccak256(bytes(signature))), ...)

abi.encodeCall(function functionPointer, (...)) returns (bytes memory)：对 functionPointer 的调用进行 ABI 编码，参数在元组中找到。执行完整的类型检查，确保类型与函数签名匹配。结果等于 abi.encodeWithSelector(functionPointer.selector, (...))



#### 字节/字符串成员

bytes.concat(...) returns  (bytes memory)     :将可变的bytes连接成一个字节数组

string.concat(...) returns  (string memory)      ：将可变的字符串连接成一个字符串数组



#### 错误处理

assert函数：若结果为否，就抛出错误，确保结果正确。（即使有错误，也会执行并扣除gas）

require：条件不满足则回滚，用于输入外部错误  （有错不会执行，也不会扣除gas）

revert ：中止执行并回滚状态更改



#### 数字和密码学函数

**addmod(uint** **x,** **uint** **y,** **uint** **k)** **returns** **(uint)** ：计算（x+y）%k

**mulmod(uint** **x,** **uint** **y,** **uint** **k)** **returns** **(uint)**   ：计算（x*y）%k

**keccak256(bytes** **memory)** **returns** **(bytes32)**   ：计算输入的 Keccak-256 哈希

**sha256(bytes** **memory)** **returns** **(bytes32)**    ：计算输入的 SHA-256 哈希

**ripemd160(bytes** **memory)** **returns** **(bytes20)**    ：计算输入的 RIPEMD-160 哈希

**ecrecover(bytes32** **hash,** **uint8** **v,** **bytes32** **r,** **bytes32** **s)** **returns** **(address)**   



#### 地址类型的成员

<img src="C:\Users\陈如意\AppData\Roaming\Typora\typora-user-images\image-20241202092749530.png" alt="image-20241202092749530"  />



**this**：当前合约的类型         **super**：访问高一级的合约

**selfdestruct(address** **payable** **recipient)**    ：销毁当前合约，将其资金发送到给定的地址类型并结束执行



#### salt选项

根据创建合约的地址、给定的盐值、被创建合约的（创建）字节码和构造函数参数计算地址。

这里的主要用例是作为**链下交互的裁判的合约，仅在发生争议时需要创建**。

```solidity
contract D{
	uint public x;
	constructor(uint a){
	x=a;
	}
}
contract{
function createDSalted(bytes32 salt, uint arg) public {
 	new D{salt:salt}(arg)
 } 
} 
```



作用域不同，可以有相同的变量名称



**public**，**internal**，**private**

**函数可见性**：**external**:只能被外部调用，无法被内部调用

​						**internal**:只能被内部调用

**payable**:用于标记可以接收以太币的函数



#### 函数修改器

**modifier**：修改器modifier 可以以声明的方式改变函数的行为。例如，你可以使用 修改器modifier 在执行函数之前自动检查条件。

```solidity
modifier onlyOwner {
        require(
            msg.sender == owner,
            "Only owner can call this function."
        );
        _;
    }
    // 该合约仅定义了一个修改器，但未使用它：它将在派生合约中使用。
    // 函数体插入在修改器定义中的特殊符号 `_;` 出现的位置。
    // 这意味着如果所有者调用此函数，则函数将被执行，否则将抛出异常。
```

如果需要访问合约C中的修改器m，可以使用C.m来引用





### 特殊函数

##### 接受以太币函数：**receive函数**：声明为

~~~solidity
receive（） external payable{...}   
~~~

该函数不能有参数，不能返回任何内容，必须具有external可见性和payable状态可变性。可以是虚拟的，可以重写，并且可以有修改器

没有接收以太币函数的合约可以作为 **coinbase 交易** （即 **矿工区块奖励**）的接收者或作为 `selfdestruct` 的目标接收以太币。

##### 回退函数：fallback函数

~~~solidity
fallback() external [payable]
fallback(bytes calldata input) external [payable] returns(bytes memory output)
//俩者均不带function关键字
~~~



##### 函数重载：同名函数但参数不同



### 事件：event

```js
var options = {
    fromBlock: 0,
    address: web3.eth.defaultAccount,
    topics: ["0x0000000000000000000000000000000000000000000000000000000000000000", null, null]
};
web3.eth.subscribe('logs', options, function (error, result) {
    if (!error)
        console.log(result);
})
    .on("data", function (log) {
        console.log(log);
    })
    .on("changed", function (log) {
});
```

```
contract ClientReceipt {
    event Deposit(
        address indexed from,
        bytes32 indexed id,
        uint value
    );
//event name(parameter，parameter，parameter);
//事件后跟emit发出

    function deposit(bytes32 id) public payable {
        // 事件通过 `emit` 发出，后跟事件的名称和（如果有的话）括号中的参数
        // 任何这样的调用（即使是深度嵌套）都可以通过
        // JavaScript API 通过过滤 `Deposit` 来检测。
        emit Deposit(msg.sender, id, msg.value);
    }
}
```



#### 自定义错误:

Solidity 中的错误提供了一种方便且节省 gas 的方式来向用户解释操作失败的原因。它们可以在合约内部和外部（包括接口和库）定义。

```solidity
error InsufficientBalance(uint256 available, uint256 required);
//assert

//revert用法
revert InsufficientBalance({
	available:balance[msg.sender],
	required:amount
});

//require用法
 require(amount <= balance[msg.sender], InsufficientBalance(balance[msg.sender], amount));
```



#### 抽象合约

```solidity
// 这些抽象合约仅用于使接口为编译器所知。注意没有主体的函数。如果合约没有实现所有函数，它只能用作接口。
abstract contract Config {
    function lookup(uint id) public virtual returns (address);  //未给出实体{}
}
```

#### 接口

类似于抽象合约，但他们不能实现任何函数

```solidity
interface Token {
    enum TokenType { Fungible, NonFungible }
    struct Coin { string obverse; string reverse; }
    function transfer(address recipient, uint amount) external;
}
```

#### 函数重写

函数可以通过继承合约进行重写，以改变其行为，如果它们被标记为 `virtual`。 重写的函数必须在函数头中使用 `override` 关键字

~~~solidity
contract Base{
	function foo()virtual external view{}
}
contract basic is Base{
	function foo()override public pure{}
}
~~~

#### 构造函数（constructor）：可在其中运行初始化代码

~~~solidity
contract name{
	uint public a;
	constructor(uint _a){
		a=_a;
	}
}
contract name1 is name{
	contructor(){}
}
~~~



### 库

~~~solidity
//库函数示例
struct Data{
	mapping(uint=>bool) flags;
}
library Set{
	function insert(Data storage data,uint value )public returns(bool){
        if (data.flags[value])
            return false ;
        else{ 
        data.flags[value]=true;
        return true;
         }
    }
}
contract RUYI{
    Data data1;
    function register(uint value)public {
        require(Set.insert(data1, value));
    }
}
~~~

#### 内联汇编（assembly）

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.16 <0.9.0;

library GetCode {
    function at(address addr) public view returns (bytes memory code) {
        assembly {
            // 获取代码的大小，这需要使用汇编
            let size := extcodesize(addr)
            // 分配输出字节数组——这也可以在没有汇编的情况下完成
            // by using code = new bytes(size)
            code := mload(0x40)
            // 新的“内存结束”，包括填充
            mstore(0x40, add(code, and(add(add(size, 0x20), 0x1f), not(0x1f))))
            // 在内存中存储长度
            mstore(code, size)
            // 真正获取代码的大小，这需要使用汇编
            extcodecopy(addr, add(code, 0x20), 0, size)
        }
    }
}
```

#### 访问外部变量、函数和库

你可以通过使用其名称访问 Solidity 变量和其他标识符。值类型的本地变量可以直接在内联汇编中使用。它们可以被读取和赋值。

对于外部函数指针，可以使用 `x.address` 和 `x.selector` 访问地址和函数选择器

对于动态 calldata 数组，你可以使用 `x.offset` 和 `x.length` 访问它们的 calldata 偏移量（以字节为单位）和长度（元素数量）

```solidity
contract C {
    // 将新的选择器和地址分配给返回变量 @fun
    function combineToFunctionPointer(address newAddress, uint newSelector) public pure returns (function() external fun) {
        assembly {
            fun.selector := newSelector
            fun.address  := newAddress
        }
    }
}
```

#### using for

指令 `using A for B` 可用于将函数（`A`）作为运算符附加到用户定义的值类型或作为任何类型（`B`）的成员函数。 成员函数接收调用它们的对象作为第一个参数（类似于 Python 中的 `self` 变量）。运算符函数接收操作数作为参数。

例如 `using {f, g as +, h, L.t} for uint`



![image-20241203150858434](C:\Users\陈如意\AppData\Roaming\Typora\typora-user-images\image-20241203150858434.png)



