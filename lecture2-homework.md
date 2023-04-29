# 第2课 课后作业

## 第1题 转换为bit位 Num2Bits
```
pragma circom 2.1.4;

template Num2Bits(nBits) {

    signal input in;
    signal output out[nBits];

    var lc1=0;
    var e2=1;
    for (var i = 0; i < nBits; i++) {
        out[i] <-- (in >> i) & 1;
        out[i] * (out[i] -1 ) === 0;
        lc1 += out[i] * e2;
        e2 = e2+e2;
    }

    lc1 === in;
}

component main { public [ in ] } = Num2Bits(8);

/* INPUT = {
    "in": "5"
} */
```

运行结果
```
out[0] = 1
out[1] = 0
out[2] = 1
out[3] = 0
out[4] = 0
out[5] = 0
out[6] = 0
out[7] = 0
```

## 第2题 判零 IsZero
```
pragma circom 2.1.4;

template IsZero() {
    signal input in;
    signal output out;

    signal inv;

    inv <-- in!=0 ? 1/in : 0;

    out <== -in*inv +1;
    in*out === 0;
}

component main { public [ in ] } = IsZero();

/* INPUT = {
    "in": "0"
} */
```

运行结果
```
out = 1
```

## 第3题 相等 IsEqual
```
pragma circom 2.1.4;

template IsZero() {

    signal input in;
    signal output out;

    signal inv;

    inv <-- in!=0 ? 1/in : 0;

    out <== -in*inv +1;
    in*out === 0;
}

template IsEqual() {
    signal input in[2];
    signal output out;

    component isz = IsZero();

    in[1] - in[0] ==> isz.in;

    isz.out ==> out;
}

component main { public [ in ] } = IsEqual();

/* INPUT = {
    "in": ["2", "2"]
} */
```

运行结果
```
out = 1
```

## 第4题 选择器 Selector
```
pragma circom 2.1.4;

template Num2Bits(n) {
    signal input in;
    signal output out[n];
    var lc1=0;

    var e2=1;
    for (var i = 0; i<n; i++) {
        out[i] <-- (in >> i) & 1;
        out[i] * (out[i] -1 ) === 0;
        lc1 += out[i] * e2;
        e2 = e2+e2;
    }

    lc1 === in;
}

template IsZero() {
    signal input in;
    signal output out;

    signal inv;

    inv <-- in!=0 ? 1/in : 0;

    out <== -in*inv +1;
    in*out === 0;
}

template IsEqual() {
    signal input in[2];
    signal output out;

    component isz = IsZero();

    in[1] - in[0] ==> isz.in;

    isz.out ==> out;
}

template LessThan(n) {
    assert(n <= 252);
    signal input in[2];
    signal output out;

    component n2b = Num2Bits(n+1);

    n2b.in <== in[0]+ (1<<n) - in[1];

    out <== 1-n2b.out[n];
}

template CalculateTotal(n) {
    signal input in[n];
    signal output out;

    signal sums[n];

    sums[0] <== in[0];

    for (var i = 1; i < n; i++) {
        sums[i] <== sums[i-1] + in[i];
    }

    out <== sums[n-1];
}

template QuinSelector(choices) {
    signal input in[choices];
    signal input index;
    signal output out;
    
    // Ensure that index < choices
    component lessThan = LessThan(4);
    lessThan.in[0] <== index;
    lessThan.in[1] <== choices;
    lessThan.out === 1;

    component calcTotal = CalculateTotal(choices);
    component eqs[choices];

    // For each item, check whether its index equals the input index.
    for (var i = 0; i < choices; i ++) {
        eqs[i] = IsEqual();
        eqs[i].in[0] <== i;
        eqs[i].in[1] <== index;

        // eqs[i].out is 1 if the index matches. As such, at most one input to
        // calcTotal is not 0.
        calcTotal.in[i] <== eqs[i].out * in[i];
    }

    // Returns 0 + 0 + 0 + item
    out <== calcTotal.out;
}


component main { public [ in, index ] } = QuinSelector(4);

/* INPUT = {
    "in": ["5","4","3","2"],
    "index": "1"
} */
```

运行结果
```
out = 4
```

## 第5题 判负 IsNegative
```
pragma circom 2.1.4;

include "circomlib/compconstant.circom";

template Sign() {
    signal input in[254];
    signal output sign;

    component comp = CompConstant(10944121435919637611123202872628637544274182200208017171849102093287904247808);

    var i;

    for (i=0; i<254; i++) {
        comp.in[i] <== in[i];
    }

    sign <== comp.out;
}

component main { public [ in ] } = Sign();

/* INPUT = {
    "in": ["1","0","0","0","1","0","0","0","1","0","0","0","1","0","0","0",
    "1","0","0","0","1","0","0","0","1","0","0","0","1","0","0","0",
    "1","0","0","0","1","0","0","0","1","0","0","0","1","0","0","0",
    "1","0","0","0","1","0","0","0","1","0","0","0","1","0","0","0",
    "1","0","0","0","1","0","0","0","1","0","0","0","1","0","0","0",
    "1","0","0","0","1","0","0","0","1","0","0","0","1","0","0","0",
    "1","0","0","0","1","0","0","0","1","0","0","0","1","0","0","0",
    "1","0","0","0","1","0","0","0","1","0","0","0","1","0","0","0",
    "1","0","0","0","1","0","0","0","1","0","0","0","1","0","0","0",
    "1","0","0","0","1","0","0","0","1","0","0","0","1","0","0","0",
    "1","0","0","0","1","0","0","0","1","0","0","0","1","0","0","0",
    "1","0","0","0","1","0","0","0","1","0","0","0","1","0","0","0",
    "1","0","0","0","1","0","0","0","1","0","0","0","1","0","0","0",
    "1","0","0","0","1","0","0","0","1","0","0","0","1","0","0","0",
    "1","0","0","0","1","0","0","0","1","0","0","0","1","0","0","0",
    "1","0","0","0","1","0","0","0","1","0","0","0","1","0"]
} */
```

运行结果
```
sign = 0
```

理解检查：为什么我们不能只使用 LessThan 或上一个练习中的比较器电路之一？

```
因为数字太大了，会溢出？
```

## 第6题 少于 LessThan
扩展 1
```
template LessThan(n) {
    assert(n <= 252);
    signal input in[2];
    signal output out;

    component n2b = Num2Bits(n+1);

    n2b.in <== in[0]+ (1<<n) - in[1];

    out <== 1-n2b.out[n];
}
```

扩展 2：编写 LessEqThan（测试 in[0] 是否 ≤ in[1]）、GreaterThan 和 GreaterEqThan
```
template LessEqThan(n) {
    assert(n <= 252);
    signal input in[2];
    signal output out;

    component n2b = Num2Bits(n+1);
    component eq = IsEqual();

    n2b.in <== in[0]+ (1<<n) - in[1];
    eq.in <== in;

    out <== 1 - (n2b.out[n]) * (1 - eq.out);
}

```
GreaterThan 和 GreaterEqThan 只需要反转LessThan和LessEqThan里边 in[0]和in[1]的顺序即可。
