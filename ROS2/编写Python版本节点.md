## 最小化Python节点
编写最小python节点代码非常简单，只需要调用一下官方的库文件即可，代码示例如下
```bash
import rclpy

from rclpy.node import Node

  

def main():

    rclpy.init();

    node = Node('python_node')

    node.get_logger().info('你好 Python 节点')

    node.get_logger().warn('你好 Python 节点')

    rclpy.spin(node)

    rclpy.shutdown()

if __name__=='__main__':

    main()
```