
## 上位机端

上位机主要负责运行AI大模型首先需要启动AI大模型

### 启动本地`Ollama`模型

```bash
 ollama run minicpm-v
```


随后到对应文件夹下启动转发脚本

### 启动转发脚本

参考命令如下：

```bash
set "OLLAMA_BASE_URL=http://127.0.0.1:11434/api/chat" && set "OLLAMA_MODEL=minicpm-v" && set "OLLAMA_KEEP_ALIVE=10m" && set "VLM_SERVICE_HOST=172.30.148.120" && set "VLM_SERVICE_PORT=19000" && python ollama_review_service.py  
```

 ​

### 状态监测

```bash
 curl.exe --max-time 5 http://172.30.148.120:19000/health  

 //预期返回结果  
 {"status":"ok"}
```


### 启动命令桥（用于激活自动巡航）

```bash
cd C:\Users\zhou\Documents\Codex\2026-04-26-files-mentioned-by-the-user-ai-2\rdk_x5_demo\notebook_vlm_service  

run_feishu_command_bridge.cmd
```


### 启动`cloudfare`

```bash
cd /d F:\Google Download  

cloudflared-windows-amd64.exe tunnel --url http://127.0.0.1:19100
```


### 上位机一键运行脚本

```bash
 # 需要在poweshell中运行  
 # 执行操作  
 # VLM review service（AI信息转发，不稳定，还要按照老方法启动）  
 # Feishu command bridge（信息转发，监听飞书端的信息）  
 # cloudflared tunnel（转发cloud端的信息）  
cd C:\Users\zhou\Documents\Codex\2026-04-26-files-mentioned-by-the-user-ai-2\rdk_x5_demo\notebook_vlm_service
  
powershell -ExecutionPolicy Bypass -File .\start_notebook_stack.ps1
```


## 开发板端

开发板端主要负责信息转发和数据读取

主要配置文件为`~/smart-care-demo`，如果需要修改接入机器人就需要修改该文件中对应地址，修改后执行`bash scripts/run_patrol_gateway.sh`使其生效

```bash
source ~/.smart_care.env  

cd ~/smart-care-demo  

bash scripts/run_patrol_gateway.sh
```


### 单开一个终端运行文件链接传输服务

```bash
 bash ~/rdk_x5_demo/scripts/run_evidence_file_server.sh  
 
 # 监测服务是否正常  
 curl http://127.0.0.1:8080/health  
 
 # 单次验证  
 source ~/.smart_care.env  
 bash ~/rdk_x5_demo/scripts/run_patrol_once.sh
```


### 快速启动服务

```bash
 # 启动服务三件套  
cd ~/rdk_x5_demo  
source ~/.smart_care.env  
bash ~/rdk_x5_demo/scripts/run_patrol_once.sh
```


### 修改后生效操作

```bash
cd ~/rdk_x5_demo/ros2_ws  
set +u  
source /opt/tros/humble/setup.bash  
colcon build  
source install/setup.bash
```



### 一键运行脚本

```bash
 # 一键自动化操作  
 # 检查并补 ~/rdk_x5_demo/config  
 # 启动 evidence_file_server  
 # 启动 patrol_gateway  
bash ~/rdk_x5_demo/scripts/start_board_stack.sh  
  
 # 查看状态  
bash ~/rdk_x5_demo/scripts/status_board_stack.sh
```


### 运行自动化脚本

运行默认脚本，其自动扫描周期是60秒，运行的脚本命令如下

```bash
bash ~/rdk_x5_demo/scripts/run_patrol_gateway.sh
```


如果想要自定义运行的时间间隔可以参考如下命令

```bash
# 当前命令为每30秒执行一次命令  
PATROL_INTERVAL_SECONDS_OVERRIDE=30 bash ~/rdk_x5_demo/scripts/run_patrol_gateway.sh

```


### 主动触发巡视脚本

```bash
source ~/.smart_care.env  
set +u  
source /opt/tros/humble/setup.bash  
source ~/rdk_x5_demo/ros2_ws/install/setup.bash  
rm -f ~/rdk_x5_demo/logs/manual_patrol.lock  
ros2 run smart_care_bridge openclaw_patrol_worker --lock-path ~/rdk_x5_demo/logs/manual_patrol.lock
```
