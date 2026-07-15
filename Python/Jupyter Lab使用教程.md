# 简介
`Jupyter Lab`是`Project Jupyter`推出的下一代基于`Web`的交互式开发环境，它是`Jupyter Notebook`的全面升级增强版

# 特征
`Jupyter Lab`虽然也是一个IDE当时它有着一些其它开发环境所没有的特性，我们可以通过下面的一组对比看出它的特性
平时在编写python代码时我们往往都是要先写好完整代码，然后再从头开始运行：
```python
a = 10
b = 20
print(a + b)
```
但是在`Jupyter Lab`中，我们可以把代码拆分成多个单元格：
```python
a = 10
b = 20
```
再单独运行下一格：
```
print(a + b)
```
对应的运行结果会直接显示在代码下方：
![](assets/Jupyter%20Lab使用教程/file-20260715171834680.png)
由于这种特性的存在，使得其特别适合：
- 学习python语法、函数、列表、字典等知识
- 验证一小段代码的运行结果
- 数据分析、机器学习训练