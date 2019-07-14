## 引入

**Beam search**是经常应用在Seq2seq任务中的解码方法, 算法本身复杂度不高, 但往往能大幅提高解码质量.

Beam search只用在**测试样本的解码过程中**, 在训练时是用不到的.

## 对比

Seq2seq任务中, 对于一个输入, 我们的任务是具有最大概率的解码序列. 可以用公式如下表示:

$$\arg \max _{y} P\left(y^{<1>}, y^{<2>}, y^{<3>}, \ldots y^{<T_{y}>} | x^{<1>}, x^{<2>}, \ldots x^{<T_{x}>}\right)$$

如果不使用任何解码方法, 在seq2seq模型中我们使用的就是**贪心搜索**(greedy search). 具体来说, 在$$t$$时刻, 根据所有的输入(encoder的输入, 来自上一个元素的输出作为当前decoder的输入), 在字典中挑选出条件概率最大的词$$y^{<t>}$$, 之后的每个时刻一次类推.

因此, 贪心算法在每个时刻, 始终是选择最大概率的词, 但这样选择出的词拼接在一起组成的序列, 往往不是上式概率最大的情况, 甚至相差很大, 而我们需要的, 正是上式概率最大的序列. 这种情况的反面例子可以参考参考资料中的第一篇的`Why not a greedy search?`部分.

如果完全从序列的角度出发, 一定能找到最优的序列, 但是字典中所有单词组成的序列数量, 是无法枚举判断得到最优序列的.

Beam search就是一种折中的方案, 在$$t$$时刻确定输出时, 会考虑之前时刻的输出序列, 并且会考虑多种前置序列, 但也使用了超参数`beam width`, 保证了搜索范围的高效.

Beam search虽然可能不会找到最优的方案, 但已经能够保证在高效的前提下, 找到接近于最优答案, 甚至就是最优答案的结果.

## 原理

具体的原理可以参考资料中第一篇的`Beam Search`部分, 以及第二篇. 这两篇文章介绍算法时都集合了具体的例子进行推导, 形象易懂.

## 代码

```python
def beam_search(self, seq, topk=3):
    if not self.decode_build:
        self._build_decode_model()

    seq = np.repeat(seq, topk, axis=0)
    encoder_output = self.encoder_model.predict_on_batch([seq])

    final_results = []
    topk_prob = np.zeros((topk,), dtype=np.float32)
    decode_tokens = [[] for _ in range(topk)]

    target_seq = np.zeros((topk, self.length_limit), dtype=np.int32)
    target_seq[:, 0] = 2

    last_k = 1

    for i in range(self.length_limit - 1):
        if last_k == 0 or len(final_results) > topk * 3:
            break  # stop conditions

        target_output = self.decoder_model.predict_on_batch([seq, target_seq, encoder_output])
        output = np.exp(target_output[:, i, :])
        output = output / np.sum(output, axis=-1, keepdims=True)
        output = np.log(output + 1e-8)  # use `log` transformation to avoid tiny probability

        candidates = []

        for k, probs in zip(range(last_k), output):
            if target_seq[k, i] == 3:
                continue

            word_p_sort = sorted(list(enumerate(probs)), key=lambda x: x[1], reverse=True)
            for ind, wp in word_p_sort[:topk]:
                candidates.append((k, ind, topk_prob[k] + wp))

        candidates = sorted(candidates,key=lambda x: x[-1], reverse=True)
        candidates = candidates[:topk]

        target_seq_bk = target_seq.copy()

        for new_k, cand in enumerate(candidates):
            k, ind, seq_p = cand
            target_seq[new_k] = target_seq_bk[k]
            target_seq[new_k, i + 1] = ind
            topk_prob[new_k] = seq_p
            decode_tokens.append(decode_tokens[k] + [self.tar_token_dict[ind]])
            if ind == 3:
                final_results.append((decode_tokens[k], seq_p))

        decode_tokens = decode_tokens[topk:]
        last_k = len(decode_tokens)

    final_results = [(x, y / (len(x) + 1)) for x, y in final_results]
    final_results = sorted(final_results, key=lambda x: x[1], reverse=True)
    return final_results
```

这是结合在seq2seq模型中的一个beam search算法的代码. Beam search算法的唯一超参数就是上文中提到的`beam width`, 也就是代码中的`topK`.

从投入开始标志`<s>`(代码对应的字典中的index为2)为起始, 逐个预测之后的每个元素. 在预测过程中, 除了需要保存现有的`topK`个候选序列(保存在变量`target_seq`中), 还要存储对应序列的整体概率(保存在`topk_prob`中).

另外需要注意的是, 代码中关于**概率计算**和`终止符</s>`的处理方法.

## 参考资料

- [Seq2Seq中的beam search算法](https://zhuanlan.zhihu.com/p/36029811?group_id=972420376412762112)
- [seq2seq中的beam search算法过程](https://zhuanlan.zhihu.com/p/28048246)
