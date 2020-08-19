# 题目描述

[剑指 Offer 20. 表示数值的字符串](https://leetcode-cn.com/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/)

请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串"+100"、"5e2"、"-123"、"3.1416"、"-1E-16"、"0123"都表示数值，但"12e"、"1a3.14"、"1.2.3"、"+-5"及"12e+5.4"都不是。

# 解题思路

标准的**有限状态机**, 定义输入的类型(每个字符的分类), 定义状态空间, 最后定义状态之间的转移方式.

定义输入类型为:

- 空格 「 」
- 数字「0—9」
- 正负号「+-」
- 小数点 「.」
- 幂符号「eE」

定义10种状态:

- 开始的空格
- 幂符号前的正负号
- 小数点前的数字
- 当小数点前为空格时，小数点
- 小数点
- 小数点后的数字
- 幂符号
- 幂符号后的正负号
- 幂符号后的数字
- 结尾的空格

状态转移关系为:

![](/Algorithm/imgs/jianzhi_20_fig1.png)

```python
from enum import Enum


class Solution:
    InputType = Enum('InputType', [
        'NUMBER',
        'POINT',
        'SIGN',
        'EXP',
        'SPACE',
        'ILLEGAL',
    ])

    State = Enum('State', [
        'INITIAL',
        'INTEGER',
        'NUM_SIGN',
        'POINT_WITHOUT_INT',
        'POINT',
        'FRACTION',
        'EXP',
        'EXP_SIGN',
        'EXP_NUMBER',
        'END',
    ])

    mapping = {
        State.INITIAL: {
            InputType.SPACE: State.INITIAL,
            InputType.NUMBER: State.INTEGER,
            InputType.SIGN: State.NUM_SIGN,
            InputType.POINT: State.POINT_WITHOUT_INT,
        },
        State.INTEGER: {
            InputType.NUMBER: State.INTEGER,
            InputType.POINT: State.POINT,
            InputType.EXP: State.EXP,
            InputType.SPACE: State.END,
        },
        State.NUM_SIGN: {
            InputType.NUMBER: State.INTEGER,
            InputType.POINT: State.POINT_WITHOUT_INT,
        },
        State.POINT_WITHOUT_INT: {
            InputType.NUMBER: State.FRACTION,
        },
        State.POINT: {
            InputType.NUMBER: State.FRACTION,
            InputType.EXP: State.EXP,
            InputType.SPACE: State.END,
        },
        State.FRACTION: {
            InputType.NUMBER: State.FRACTION,
            InputType.EXP: State.EXP,
            InputType.SPACE: State.END,
        },
        State.EXP: {
            InputType.SIGN: State.EXP_SIGN,
            InputType.NUMBER: State.EXP_NUMBER,
        },
        State.EXP_SIGN: {
            InputType.NUMBER: State.EXP_NUMBER,
        },
        State.EXP_NUMBER: {
            InputType.NUMBER: State.EXP_NUMBER,
            InputType.SPACE: State.END,
        },
        State.END: {
            InputType.SPACE: State.END,
        },
    }

    def match(self, char):
        if char.lower() == 'e':
            return self.InputType.EXP
        elif char == ' ':
            return self.InputType.SPACE
        elif char in ('-', '+'):
            return self.InputType.SIGN
        elif char == '.':
            return self.InputType.POINT
        elif char in (str(t) for t in range(10)):
            return self.InputType.NUMBER
        else:
            return self.InputType.ILLEGAL

    def isNumber(self, s: str) -> bool:
        state = self.State.INITIAL
        for c in s:
            ctype = self.match(c)
            if ctype not in self.mapping[state]:
                return False
            state = self.mapping[state][ctype]
        return state in [self.State.INTEGER, self.State.POINT, self.State.FRACTION, self.State.EXP_NUMBER, self.State.END]

```
