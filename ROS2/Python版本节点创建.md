## 创建指令
```bash
ros2 pkg create --build-type ament_python --license Apache-2.0 demo_python_pkg
```
使用当前指令即可创建一个新的节点

### 命令解析
1. `ros2 pkg create`
	- `ros2`：`ROS 2`命令行工具
	- `pkg`：操作功能包
	- `create`：创建一个新功能包
	因此这前一段命令的整体意思就是：“创建一个新的`ROS 2`功能包
2. `--build-type ament_python`
	- `ament_python`：构建`python`包
	指定构建包的类型为`python`版本
3. `--license Apache-2.0`
	这个选项是在指定功能包的证书
## 创建结果



