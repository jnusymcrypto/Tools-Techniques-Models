# Minizinc + SYM-Cryptanalysis 语法指南

针对不同求解器, 语法会有所不同.

下列为对称密码分析常用的语法, 并不包含所有.



## CP-SAT

例子: [Demo.mzn](https://github.com/SymCNote/Tools-Techniques-Models/blob/main/codes/Demo.mzn), [para.dzn](https://github.com/SymCNote/Tools-Techniques-Models/blob/main/codes/para.dzn)

### 1. 参数 (常量/变量*) 定义

**注意:** 

1. *"常量参数" 必须在运行前赋值, 可以外部传参或设置默认值.*
2. *CP-SAT 不支持 float 变量*



#### 常量参数

* **int-eger**

  ```
  % integer parameters
  int: INT_1;              % 必须外部传入
  int: INT_2 = 128;        % 有默认值，不可被覆盖
  % ---
  output[show(INT_1) ++ "\n"];
  output[show(INT_2) ++ "\n"];
  ```

* **bool**

  <u>无法自动类型转换</u>, 即 `true` $\nleftrightarrow$ $1$.

  ```
  % bool parameters
  bool: BOOL_1;
  bool: BOOL_2 = true;
  % ---
  output[show(BOOL_1) ++ "\n"];
  output[show(BOOL_2) ++ "\n"];
  ```

  传参方式和 int-eger 相同

* **list**

  1. <u>数组内部为常量参数时, 需要运行前赋值, 支持推导赋值</u>
  2. 数组下标默认从 1 开始, 如需重新定义, 需要用 `array1d(0..2,[0,1,2])` (对 dim-1 数组)

  ```
  % list 
  array[1..3] of 0..3: LIST_1;                           % 默认数组下标从 1 开始    
  array[0..2] of int: LIST_2;                            % 传入格式为 array1d(0..2,[1,2,3])
  array[0..2, 1..3] of int: LIST_3 = array2d(0..2,1..3,[3,2,1, 6,5,4, 9,8,7]);    % 传入格式为 array1d(0..2,[1,2,3])
  % ---
  output[show(LIST_1) ++ "\n"];
  output[show(LIST_2) ++ "\n"];
  output[show(LIST_3) ++ "\n"];
  ```

* **set** (很少用)

  1. 集合用的比较少, 但可以在一些场景尝试用其代替 list
  2. 集合使用时: 无序, 去重, 支持集合推导/运算, 内存访问友好
  3. 其余同上 list

  ```
  % set 
  set of 0..7: SET_1 = 0..7;
  % 支持集合推导式
  set of int: EVEN = {i | i in SET_1 where i mod 2 = 0};
  set of int: POWER2 = {2^i | i in 0..7};
  % 支持集合运算
  set of 0..255: UNION = EVEN union POWER2;
  set of 0..255: INTERSECT = EVEN intersect POWER2;
  % ---
  output[show(SET_1) ++ "\n"];
  output[show(EVEN) ++ "\n"];
  output[show(POWER2) ++ "\n"];
  output[show(UNION) ++ "\n"];
  output[show(INTERSECT) ++ "\n"];
  ```

**传参方式:**

外部传参方式：

1. 命令行: `minizinc .\Demo.mzn -D "X=10"`

2. 外部文件, 需要有 "para.dzn"

   ```
   % para.dzn
   
   X = 10;
   ```

   同时需要在主文件引入外部文件 `include para.dzn`



#### $\star$ 决策变量

变量声明变为: `int/bool` $\rightarrow$ `var int/bool/"domine"`, 在 list 中也是如此.

```
int: INTa = 7;
var int: I_1;                           % 决策变量，整数空间
var 0..255: I_2;                        % 决策变量，指定整数空间
var bool: B;                            % 决策变量，布尔空间
array [0..2] of var int: VAR_LIST;      % 决策变量，数组
array [0..2] of var 0..2: VAR_LIST_2;   % 决策变量，指定空间数组
```

---

### 2. 方法

这个例子下面用

```
int: R = 2;
int: C = 8;
array [0..R] of var int: X;
array [0..R, 0..C] of var int: Y;
```

其中最常用的为 `forall` `exists` `sum` `max/min` `if-then-else-endif`

* **全称量词 `forall`**

  ```
  forall(i in 迭代范围)(约束语句);
  ```

  ```
  constraint forall(i in 0..R)(X[i] <= 100);
  constraint 
  	forall(i in 0..R)(
  		forall(j in 0..C)(
  			Y[i,j] <= 100
  		)
  	)
  ;
  ```

* **存在量词 `exists`**

  声明：一个列表/集合是否存在某个值 (也可用于判断)

  ```
  exists(i in 迭代范围)(约束语句);
  ```

  ```
  constraint exists(i in 0..R)(X[i] == 0);
  constraint if exists(i in 0..R)(X[i] == 0) then exists(i in 0..R)(X[i] == 1) else exists(i in 0..R)(X[i] == 2) endif;
  ```

* **求和 `sum`**

  声明：列表求和 = 某个值

  ```
  sum(i in 迭代范围)(求和变量)=SUM;
  ```

  ```
  constraint sum(i in 0..R)(X[i]) = 5;
  ```

* **最大/最小 `max/min`**

  ```
  constraint max(i in 0..R)(X[i]) <= 3;
  constraint min(i in 0..R)(X[i]) >= 2;
  ```

* **求积 `product` (很少用)**

  声明：求列表内的所有元素乘积

  ```
  var int: prod; % 这样写会直接打印出 prod，也可以 prod = product(...)不会打印
  constraint prod = product(i in 0..R)(X[i]);
  ```

* **条件表达式 `if-then-else-endif`**

  标准语法: `if AAA then BBB else CCC endif;`

  正向判定法: `if AAA then BBB endif;` 这种写法会使不满足 `AAA` 的数据取值随机化

* **表约束 `table`**

  要求值 `x` 必须来自表 `table`

  ```
  table(变量x, 表table);
  ```

  ```
  array[1..3] of var bool: x;
  array[1..4, 1..3] of int: truth_table = array2d(1..4, 1..3, [
      0, 0, 0,
      0, 1, 1,
      1, 0, 1,
      1, 1, 0
  ]);
  constraint table(x, truth_table); % 要求 x 必须由 truth_table 中取值
  constraint x[2] = x[3] /\ x[1] != x[2];
  ```

  习惯不用上面这种方式, 用下面这种 (替换最后两句):

  ```
  constraint x[1] = truth_table[2,1] /\ x[2] = truth_table[2,2] /\ x[3] = truth_table[2,3];
  ```

* **列表推导/计算**

  * 列表推导生成：

    ```
    array[1..R] of var int: squares;
    constraint squares = [i*i | i in 1..R];
    ```

  * 上述涉及遍历数组的操作方法均可转换为对数组本身的操作, 如:

    ```
    var int: MAX;
    constraint MAX = max([X[i] | i in 0..R] ++ [Y[m,n] | m in 0..R, n in 0..C]);
    ```

### * 谓词

谓词 `predicate` 类似 python 中的方法, 语法为:

```
predicate predicate_name(参数1, 参数2, ...) = 约束表达式;
```

```
var int: x;
constraint in_range(x,2,3);

predicate in_range(var int: x, int: low, int: high) = 
    (x >= low) /\ (x <= high)
;
```

---

### 3. 运算符

`and = /\`

`or = \/`

`not = not(clause)`  (少用)

` mod = (clause) mod N` (注意括号)

蕴含: `->`   `constraint (X[1] > 0) -> (X[2] < 10);`

等价: `<->`   `constraint (X[1] > 0) <-> (X[2] < 10);`

*建议在逻辑运算符外均添加括号 (优先级目前不确定)*

---

### 4. 求解

```
solve minimize objective;
solve maximize probability;
solve satisfy;
```

可用 `weight1*obj1 + weight2*obj2` 求多目标



命令行调用求解:

```
minizinc --solver CP-SAT[求解器] -p 16[线程数] -D 'X=1'[传入参数] 'Model.mzn'[求解模型] 'Result,txt'[输出文件]
```

```
minizinc --solver CP-SAT -p 8 -D "INT_0 = 0" ".\Demo.mzn" > "res.txt"
```

---

### 5. 补充

 临时变量 `let`

```
let {
    % 局部变量定义
    <定义1>;
    <定义2>;
    ...
} in <表达式>
```

```
var 1..10: x = 5;
var 1..10: y = 3;

var int: result = 
    let {
        var int: add = x + y;
        var int: minus = x - y;
    } in add * minus;

output ["Result: ", show(result), "\n"];
```

在迭代中也可使用该临时变量, 但需注意 *临时变量 无法被打印出, 很难 debug.*
