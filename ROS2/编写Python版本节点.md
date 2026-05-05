## 最小化Python节点
编写最小python节点代码非常简单，只需要调用一下官方的库文件即可，代码示例如下
```python
# 导入核心ROS库
import rclpy

from rclpy.node import Node

  
def main():
	# 初始化底层的ROS2 API
    rclpy.init()
	# 创建新节点
    node = Node('python_node')
	# 输出普通信息
    node.get_logger().info('你好 Python 节点')
	# 输出警告信息
    node.get_logger().warn('你好 Python 节点')
	# 控制节点进入死循环，保证节点一直存活
    rclpy.spin(node)
	# 起立节点占用的资源
    rclpy.shutdown()

if __name__=='__main__':

    main()
```
上述最小代码可以用于检测`ROS2`环境配置是否正确