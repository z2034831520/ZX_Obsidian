## 简介
`lambda`表达式用于创建一个简单的匿名函数：函数没有正式名称，通常只执行一条很短的表达式，相较于传统函数，lambda表达式可以通过寥寥几笔就能实现一个简单的函数逻辑

## 格式
`lambda 参数: 表达式`
通过一个简单的对比我们就能够直观的感受到二者的差距
### 传统写法
```python
def add(x,y):
	return x + y
print(add(2,3))
```
### `lambda`写法(没有函数名，直接返回计算结果)
```python
add = lambda x, y: x + y

print(add(2,3))
```

