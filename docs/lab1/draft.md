# 实验一草稿

## 目标

Java语言词法分析器：

```java
/**
 * class Test
 * 用于演示词法分析器的多行注释
 * 注意：注释会被忽略
 */
public class Test{
	private int a;
	protected double b = 0.1;
	final public String c = "Test";
    // 演示单行注释
    static Test instance;
    
    Test(){
    	this(0);	// 演示语句后注释
    }
    
    private Test(int a){
    	this.a = a;
    }

	public void func1(){
		System.out.println("func1");
	}
	
	protected int func2(int a, int b){
		return a + b;
	}
	
	private String func3(/* 演示行内注释 */){
		return "Test";
	}
	
	public static Test getInstance(){
		if(instance == null){
			instance = new Test();
		}
		return instance;
	}  

	public static void main(String[] args){
		Test test1 = new Test();
		Test test2 = Test.getInstance();
		if(test1 == test2){
			System.out.println("test1 == test2");
		}else{
			System.out.println("test1 != test2");
		}
		while(true){}
	}
}
```

## REs

- 保留字：

  ```java
  abstract, assert,
  
  boolean, break, byte,
  
  case, catch, char, class, const, continue,
  
  default, do, double,
  
  else, enum, extends,
  
  final, finally, float, for,
  
  goto,
  
  if, implements, import, instanceof, int, interface,
  
  long,
  
  native, new,
  
  package, private, protected, public,
  
  return,
  
  short, static, strictfp, super, switch, synchronized,
  
  this, throw, throws, transient, try,
  
  void, volatile,
  
  while
  ```

  这个不需要使用专门的RE，只需要在提取出类名、函数名或者变量名之后来判断是否在这些保留字中即可

- 运算符和界符

  - 运算符（operator）（?:这种三目运算符就拆分成?和:两个终结符）

    ps：为了降低复杂度，对于有歧义的表达式如`a+++++a`这种，直接报错

    ```java
    ++, --, +, -, ~, !, *, /, %, <<, >>, >>>, <, >, <=, >=, ==, !=, &, ^, |, &&, ||, =, +=, -=, *=, /=, %=, &=, ^=, |=, <<=, >>=, >>>=, ., @, ?, :
    ```

  - 界符（delimiter）

    ```java
    (, ), {, }, [, ], ;, \,(逗号运算符)
    ```

  这些是终结符

- 类名、函数名、变量名等（暂时不支持非ASCII字符）

  _、$、英文字符开头（[a-zA-Z]），并由\_、\$、数字（[0-9]）和英文字母（[a-zA-Z]）组成

  RE：$variable=>[\_\backslash\$a-zA-Z][\_\backslash\$a-zA-Z0-9]* $

- 字符串

  RE：string -> "([ ^ "\\\\] | (\\\\"))*"​

  这里的`.`实际表示的是非`"`的任意字符

- 字符 

  RE：character -> '( [ ^ \\\\' ] | \\\\[0-9btnfr\\\\'] )'​

- 数字

  由数字和`.`组成，并最多含有一个`.`

  或者由0x开头，由[0-9a-fA-F]组成

  文法：

  $number=>float|integer$

  $float=>.digit+|digit+.digit+|digit+.$

  - 对于float的文法，可以用来直接构造DFA：

    $S=>digit\;B|.A$

    $B=>digit\;B|.C$

    $A=>digit\;C$

    $C=>digit\;C|\epsilon$

  $integer=>dec|hex$

  $dec=>digit+$

  $hex=>0(x|X)(digit|[a-fA-F])+$

  $digit=>[0-9]$

  因此RE是：

  $number=>[0-9]+|0(x|X)[0-9a-fA-F]+|\backslash.[0-9]+|[0-9]+.[0-9]+|[0-9+].$

## NFAs

- 类名、函数名、变量名等

  ![](https://i.loli.net/2019/12/05/CYH4zQ9UofwcaNK.jpg)

- 字符串

  ![](https://i.loli.net/2019/12/05/irbyheg39TO12Az.jpg)

- 字符

  ![](https://i.loli.net/2019/12/06/clG7WkzfMj3QxNL.png)

- 数字

  ![](https://i.loli.net/2019/12/05/76mGiSYxhKWeBAb.jpg)

  其实将浮点数、整数（十进制和十六进制）混在一起不太好，上面的图就不删了，下面将它们分开：

  - 浮点数

    ![](https://i.loli.net/2019/12/05/mHs2M1Cl8G9xd4a.jpg)

  - 十进制整数

    ![](https://i.loli.net/2019/12/05/1b3eoFMyGPOUtwd.jpg)

  - 十六进制整数

    ![](https://i.loli.net/2019/12/05/dsFCkMy3j6o1iVr.jpg)



## aNFA

这一步是将上面的NFAs合并成一个大的NFA

~~（图过大，这一步先省略了。。。）~~

题外话：本来跳过了和并NFAs这一步，直接将各个NFA化简成了DFAo，然后就开始写代码，直到写完Lex框架后发现，在遍历输入字符流时容易出现不知道选择那个DFAo继续进行的情况，不得不通过试探法来尝试选择最优的DFAo，如下：

```java
Lex lex = new Lex();
char cur;
StringBuilder word = new StringBuilder();
while (charReader.hasNext()) {
    cur = charReader.next();
    word.append(cur);
    if (cur == '_' || cur == '$' || CharUtil.isAlphabet(cur)) {
        // 类名，函数名或者变量名，使用NameDFA
    } else if (cur == '\"') {
        // 字符串，使用StringDFA
    } else if (cur == '\'') {
        // 字符，使用CharacterDFA
    } else if (cur == '.') {
        // 可能有两种情况，
        // （1）以.开头的浮点数
        // （2）运算符.
    }else if (CharUtil.isDigit(cur)){
        // 可能有三种情况
        // （1）浮点数
        // （2）十进制整数
        // （3）十六进制整数
    }else {
        // 运算符或界符
    }
}
```

这不就是另一种的NFA？即同一条边有多个后续状态，这样不行！！于是我决定补上这一步，但是基于优化后的DFAo

- 首先合并浮点数<u>（由于以`.`开头的浮点数可能会与运算符`.`产生歧义，因此给浮点数的DFAo的I2设置为终态，以表示运算符`.`）</u>、十进制整数和十六进制整数

  | I              | [0-9]    | .      | 0      | x\|X    | [0-9a-fA-F] |
  | -------------- | -------- | ------ | ------ | ------- | ----------- |
  | I0={0,1,2,3,7} | {4,8}=I1 | {5}=I2 | {9}=I3 |         |             |
  | I1={4,8}       | {4,8}=I1 | {6}=I4 |        |         |             |
  | I2={5}         | {6}=I4   |        |        |         |             |
  | I3={9}         |          |        |        | {10}=I5 |             |
  | I4={6}         | {6}=I4   |        |        |         |             |
  | I5={10}        |          |        |        |         | {11}=I6     |
  | I6={11}        |          |        |        |         | {11}=I6     |

  ![](https://i.loli.net/2019/12/06/glVQo4DfZeRiCbq.jpg)

  其中：

  - I1：十进制整数
  - I2：运算符`.`
  - I4：浮点数
  - I6：十六进制整数

  （**后来发现上面的图还是NFA，修正图如下**）

- 然后将剩余的三个DFA和上面的合并

  ![](https://i.loli.net/2019/12/06/gOSTH6zDdsX8t7P.png)

## DFA

将合并之后的NFA转换成DFA

（这图比较大，手画比较麻烦，就直接将NFAs转成DFAs得了orz）

- 类名、函数名、变量名等

  ![](https://i.loli.net/2019/12/05/eXzgZbQmEO9STY6.jpg)

- 字符串

  （此图的I3->I4有点错误，边上应该为：["btnrf\\\\']）

  ![](https://i.loli.net/2019/12/05/Cks5Kb2NFVQLfW8.jpg)

- 字符（已经是DFA）

- 数字

  - 浮点数

    | I                   | [0-9]                      | .                  |
    | ------------------- | -------------------------- | ------------------ |
    | I0={x}              | {4,10}={4,5,6,10,11,12}=I1 | {1}={1}=I2         |
    | I1={4,5,6,10,11,12} | {5,11}={5,6,11,12}=I3      | {7,y}={7,8,9,y}=I4 |
    | I2={1}              | {2}={2,3,y}=I5             |                    |
    | I3={5,6,11,12}      | {5,11}=I3                  | {7,y}=I4           |
    | I4={7,8,9,y}        | {8,y}={8,9,y}=I6           |                    |
    | I5={2,3,y}          | {3}={3,y}=I7               |                    |
    | I6={8,9,y}          | {8,y}=I6                   |                    |
    | I7={3,y}            | {3}=I7                     |                    |
    |                     |                            |                    |

    ![](https://i.loli.net/2019/12/05/KAljaFOIPUMxw2N.jpg)

  - 十进制整数

    | I          | [0-9]           |
    | ---------- | --------------- |
    | I0={x}     | {1}=>{1,2,y}=I1 |
    | I1={1,2,y} | {2}=>{2,y}=I2   |
    | I2={2,y}   | {2}=I2          |

    ![](https://i.loli.net/2019/12/05/4osjzJvNE2OkUhI.jpg)

  - 十六进制整数

    | I          | [0-9a-fA-F]  | 0      | x\|X   |
    | ---------- | ------------ | ------ | ------ |
    | I0={x}     |              | {1}=I1 |        |
    | I1={1}     |              |        | {2}=I2 |
    | I2={2}     | {3,4,y}=I3   |        |        |
    | I3={3,4,y} | {4}={4,y}=I4 |        |        |
    | I4={4,y}   | {4}=I4       |        |        |
    |            |              |        |        |

    ![](https://i.loli.net/2019/12/05/fkNlAOSG9QzpmZ8.jpg)

## DFAo

优化DFA，使得状态数最少

- 类名、函数名、变量名等

  已经是DFAo

- 字符串

  已经是DFAo

- 字符

  已经是DFAo

- 数字

  - 浮点数

    ![](https://i.loli.net/2019/12/05/6rBWQid4hyTYMqX.jpg)

  - 十进制整数

    ![](https://i.loli.net/2019/12/05/3VCSikQ4BljYog1.jpg)

  - 十六进制整数  

    ![](https://i.loli.net/2019/12/05/woUnIx6TGX49keL.jpg)

































