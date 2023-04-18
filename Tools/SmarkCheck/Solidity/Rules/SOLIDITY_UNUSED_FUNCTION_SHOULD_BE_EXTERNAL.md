# Analysis of Smart Contract Security Vulnerabilities and Tools ![](https://img.shields.io/badge/-Live-brightgreen)
![](https://img.shields.io/badge/Batch-20CYS-green) ![](https://img.shields.io/badge/Batch-UG21CYS-lightgreen) ![](https://img.shields.io/badge/Batch-PG21CYS-green) ![](https://img.shields.io/badge/Batch-UG22CYS-lightgreen) ![](https://img.shields.io/badge/Batch-PG21CYS-green) ![](https://img.shields.io/badge/Batch-PhD-darkgreen) ![](https://img.shields.io/badge/-B_RIG-darkgreen)<br/>   ![](https://img.shields.io/badge/BlockchainCourse-20CY712-green)  ![](https://img.shields.io/badge/-M.Tech_Dissertation-blue) ![](https://img.shields.io/badge/Focus-Smart_Contract_Security-yellow) <br/>
![](https://img.shields.io/badge/Blockchain-Ethereum-blue)   <br/> 
![](https://img.shields.io/badge/Language-Solidity-blue)

## SOLIDITY_UNUSED_FUNCTION_SHOULD_BE_EXTERNAL

![](https://img.shields.io/badge/Pattern_ID-73ufc1-gold) ![](https://img.shields.io/badge/Severity-1-brown) 

```
(contractDefinition | libraryDefinition)//functionDefinition
                        <!-- Looks for public visibility -->
                        [visibleType[text()[1] = "public"]]
                        <!-- check that there are no function calls in contract -->
                        [identifier
                            [not(text()[1]
                                = ancestor::sourceUnit//functionCall/functionName/identifier/text()[1])
                            ]
                        ]
```

![](https://img.shields.io/badge/Pattern_ID-901eae-gold) ![](https://img.shields.io/badge/Severity-1-brown) 

```
functionCall
                        [functionName/identifier[text()[1] = "keccak256"]]
                        [callArguments/tupleExpression[count(expression) > 1]]
```

![](https://img.shields.io/badge/Pattern_ID-91h3sa-gold) ![](https://img.shields.io/badge/Severity-1-brown) 

```
functionFallBackDefinition/visibleType[text()[1] != "external"]
```

![](https://img.shields.io/badge/Pattern_ID-341gim-gold) ![](https://img.shields.io/badge/Severity-1-brown) 

```
functionDefinition
                        [not(visibleType) or visibleType[text()[1] != "external"]]
                        //parameter
                            [
                                typeName[text()[1] = "[]"]
                                or typeName/elementaryTypeName
                                    [text()[1] = "bytes" or text()[1] = "string"]
                                or typeName//identifier
                                    [text()[1]
                                        = ancestor::contractDefinition//structDefinition/identifier/text()[1]
                                    ]
                            ]
                            [not(storageLocation)]
```

![](https://img.shields.io/badge/Pattern_ID-441gim-gold) ![](https://img.shields.io/badge/Severity-1-brown) 

```
functionDefinition
                        //variableDeclaration
                            [
                                typeName[text()[1] = "[]"]
                                or typeName/elementaryTypeName
                                    [text()[1] = "bytes" or text()[1] = "string"]
                                or typeName//identifier
                                    [text()[1]
                                        = ancestor::contractDefinition//structDefinition/identifier/text()[1]
                                    ]
                            ]
                            [not(storageLocation)]
```

### Sample Code

```
pragma solidity 0.6.0;

abstract contract T {
    function f0(uint256 a) pure external virtual returns (uint256);
}

contract Test is T {

    // <yes> <report> SOLIDITY_UNUSED_FUNCTION_SHOULD_BE_EXTERNAL 73ufc1
    function badf1() public pure returns (uint256){
        return 5;
    }

    // <yes> <report> SOLIDITY_UNUSED_FUNCTION_SHOULD_BE_EXTERNAL 73ufc1
    function badf2(uint256 a) pure public returns (uint256){
        require(a > 7);
        return 5;
    }

    function notbadf2(uint256 a) pure public returns (uint256){
        require(a > 7);
        return 5;
    }

    function f0(uint256 a) pure external override returns (uint256){
        require(a > 7);
        return 5;
    }

    function f1() private pure returns (uint256){
        return 5;
    }

    function f2(uint256 a) private pure returns (uint256){
        require(a > 7);
        return 5;
    }

    function f3() public pure returns (uint256){
        return 5;
    }

    function f4(uint256 a) public pure returns (uint256){
        require(a > 7);
        return 5;
    }

    function useFunctions() private returns(uint256) {
        uint256 temp = f3();
        temp = f4(9);
        return temp;
    }
}

contract Child is Test {
    function useOne() private {
        notbadf2(54);
    }
}

interface ITest {
    function notUsed() public;
}
```