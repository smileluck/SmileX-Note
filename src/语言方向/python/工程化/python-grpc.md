## python集成grpc

### 安装和使用

1. 安装grpc
```bash
# 安装 gRPC 核心库
pip install grpcio

# 安装 gRPC 工具（包含 protoc 编译器和代码生成插件）
pip install grpcio-tools
```

2. 批量创建proto文件对应python脚本
   
```python
import os
import subprocess

def batch_generate_grpc(proto_dir, output_dir=None):
    """
    批量生成指定目录下所有.proto文件的Python代码
    
    :param proto_dir: 存放.proto文件的目录
    :param output_dir: 生成代码的输出目录，默认与proto文件同目录
    """
    # 确保输出目录存在
    if output_dir and not os.path.exists(output_dir):
        os.makedirs(output_dir, exist_ok=True)
    
    # 遍历目录下所有.proto文件
    for root, _, files in os.walk(proto_dir):
        for file in files:
            if file.endswith('.proto'):
                proto_path = os.path.join(root, file)
                print(f"处理文件: {proto_path}")
                
                # 确定输出目录
                current_output_dir = output_dir if output_dir else root
                
                # 构建命令
                cmd = [
                    'python', '-m', 'grpc_tools.protoc',
                    f'-I{proto_dir}',  # proto文件的根目录
                    f'--python_out={current_output_dir}',  # 生成消息类
                    f'--grpc_python_out={current_output_dir}',  # 生成gRPC代码
                    proto_path
                ]
                
                # 执行命令
                result = subprocess.run(cmd, capture_output=True, text=True)
                
                if result.returncode == 0:
                    print(f"成功生成 {file} 的代码")
                else:
                    print(f"生成 {file} 代码失败:")
                    print(f"错误输出: {result.stderr}")

if __name__ == "__main__":
    # 示例使用
    PROTO_DIRECTORY = "./protos"  # 存放所有.proto文件的目录
    OUTPUT_DIRECTORY = "./generated"  # 生成代码的输出目录
    
    batch_generate_grpc(PROTO_DIRECTORY, OUTPUT_DIRECTORY)
    print("批量生成完成")
    
```

3. 基类
```python
import grpc
import time
from typing import Callable, Dict, Any, Optional, List
from concurrent import futures


class GRPCClientBase:
    """gRPC客户端基础类，封装连接管理和通用逻辑"""
    
    def __init__(self, server_address: str, max_retry: int = 3, retry_delay: float = 1.0):
        """
        初始化gRPC客户端
        
        :param server_address: gRPC服务器地址，如"localhost:50051"
        :param max_retry: 最大重试次数
        :param retry_delay: 重试延迟时间(秒)
        """
        self.server_address = server_address
        self.max_retry = max_retry
        self.retry_delay = retry_delay
        self.channel = None
        self.stub = None
        self._initialize_channel()
        self._initialize_stub()
    
    def _initialize_channel(self) -> None:
        """初始化gRPC通道"""
        self.channel = grpc.insecure_channel(self.server_address)
        
        # 验证连接
        try:
            grpc.channel_ready_future(self.channel).result(timeout=5)
        except grpc.FutureTimeoutError:
            raise ConnectionError(f"无法连接到gRPC服务器: {self.server_address}")
    
    def _initialize_stub(self) -> None:
        """初始化gRPC存根，需在子类中实现"""
        raise NotImplementedError("子类必须实现_initialize_stub方法")
    
    def _retry_wrapper(self, func: Callable, *args, **kwargs) -> Any:
        """
        重试装饰器，处理临时错误
        
        :param func: 要执行的gRPC调用函数
        :return: 函数返回值
        """
        for attempt in range(self.max_retry):
            try:
                return func(*args, **kwargs)
            except (grpc.RpcError, ConnectionError) as e:
                if attempt == self.max_retry - 1:
                    raise  # 最后一次尝试失败则抛出异常
                print(f"调用失败，将在{self.retry_delay}秒后重试 (尝试 {attempt + 1}/{self.max_retry})")
                time.sleep(self.retry_delay)
    
    def close(self) -> None:
        """关闭gRPC通道"""
        if self.channel:
            self.channel.close()
            self.channel = None
            self.stub = None
    
    def __enter__(self) -> "GRPCClientBase":
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb) -> None:
        self.close()


class GRPCServerBase:
    """gRPC服务端基础类，封装服务启动和管理逻辑"""
    
    def __init__(self, host: str = "0.0.0.0", port: int = 50051, max_workers: int = 10):
        """
        初始化gRPC服务器
        
        :param host: 服务器主机地址
        :param port: 服务器端口
        :param max_workers: 最大工作线程数
        """
        self.host = host
        self.port = port
        self.server_address = f"{host}:{port}"
        self.max_workers = max_workers
        self.server = None
        self._services = []
    
    def _register_services(self) -> None:
        """注册gRPC服务，需在子类中实现"""
        raise NotImplementedError("子类必须实现_register_services方法")
    
    def start(self, block: bool = True) -> None:
        """
        启动gRPC服务器
        
        :param block: 是否阻塞当前线程
        """
        # 创建服务器
        self.server = grpc.server(futures.ThreadPoolExecutor(max_workers=self.max_workers))
        
        # 注册服务
        self._register_services()
        
        # 绑定地址并启动
        self.server.add_insecure_port(self.server_address)
        self.server.start()
        print(f"gRPC服务器已启动，地址: {self.server_address}")
        
        if block:
            try:
                while True:
                    time.sleep(3600)  # 保持服务器运行
            except KeyboardInterrupt:
                self.stop()
    
    def stop(self, grace: Optional[float] = None) -> None:
        """
        停止gRPC服务器
        
        :param grace: 优雅关闭的时间(秒)
        """
        if self.server:
            print(f"正在关闭gRPC服务器...")
            self.server.stop(grace)
            self.server = None
            print("gRPC服务器已关闭")
    
```
4. 示例服务
proto文件
```proto
syntax = "proto3";

package helloworld;

// 定义请求消息
message HelloRequest {
  string name = 1;
  int32 age = 2;
}

// 定义响应消息
message HelloReply {
  string message = 1;
  int64 timestamp = 2;
}

// 定义gRPC服务
service Greeter {
  // 简单RPC方法
  rpc SayHello (HelloRequest) returns (HelloReply) {}
  
  // 服务端流式RPC
  rpc SayHelloStream (HelloRequest) returns (stream HelloReply) {}
}

```
客户端实现
```python
import time
import helloworld_pb2
import helloworld_pb2_grpc
from grpc_base import GRPCClientBase


class HelloWorldClient(GRPCClientBase):
    """HelloWorld服务的gRPC客户端实现"""
    
    def _initialize_stub(self) -> None:
        """初始化HelloWorld服务存根"""
        self.stub = helloworld_pb2_grpc.GreeterStub(self.channel)
    
    def say_hello(self, name: str, age: int) -> dict:
        """
        调用SayHello RPC方法
        
        :param name: 姓名
        :param age: 年龄
        :return: 响应字典
        """
        request = helloworld_pb2.HelloRequest(name=name, age=age)
        
        # 使用重试机制调用RPC
        response = self._retry_wrapper(
            self.stub.SayHello,
            request,
            timeout=5  # 设置超时时间
        )
        
        return {
            "message": response.message,
            "timestamp": response.timestamp
        }
    
    def say_hello_stream(self, name: str, age: int) -> List[dict]:
        """
        调用SayHelloStream流式RPC方法
        
        :param name: 姓名
        :param age: 年龄
        :return: 响应列表
        """
        request = helloworld_pb2.HelloRequest(name=name, age=age)
        responses = []
        
        try:
            # 处理流式响应
            for response in self.stub.SayHelloStream(request, timeout=10):
                responses.append({
                    "message": response.message,
                    "timestamp": response.timestamp
                })
            return responses
        except grpc.RpcError as e:
            print(f"流式调用失败: {e}")
            raise


# 使用示例
if __name__ == "__main__":
    try:
        # 使用上下文管理器自动管理连接
        with HelloWorldClient("localhost:50051") as client:
            # 调用简单RPC
            result = client.say_hello("Alice", 30)
            print(f"简单RPC响应: {result}")
            
            # 调用流式RPC
            stream_results = client.say_hello_stream("Bob", 25)
            print("流式RPC响应:")
            for res in stream_results:
                print(res)
    except Exception as e:
        print(f"客户端错误: {e}")

```
服务端
```python
import time
import helloworld_pb2
import helloworld_pb2_grpc
from grpc_base import GRPCServerBase


class GreeterService(helloworld_pb2_grpc.GreeterServicer):
    """HelloWorld服务实现"""
    
    def SayHello(self, request, context):
        """实现SayHello RPC方法"""
        message = f"Hello, {request.name}! You are {request.age} years old."
        return helloworld_pb2.HelloReply(
            message=message,
            timestamp=int(time.time())
        )
    
    def SayHelloStream(self, request, context):
        """实现SayHelloStream流式RPC方法"""
        for i in range(5):  # 发送5条流式响应
            message = f"Hello, {request.name}! This is message {i+1}"
            yield helloworld_pb2.HelloReply(
                message=message,
                timestamp=int(time.time())
            )
            time.sleep(1)  # 模拟处理延迟


class HelloWorldServer(GRPCServerBase):
    """HelloWorld服务的gRPC服务器实现"""
    
    def _register_services(self) -> None:
        """注册HelloWorld服务"""
        helloworld_pb2_grpc.add_GreeterServicer_to_server(
            GreeterService(),
            self.server
        )


# 启动服务器
if __name__ == "__main__":
    server = HelloWorldServer(port=50051)
    try:
        server.start()  # 阻塞运行
    except KeyboardInterrupt:
        server.stop()
```
