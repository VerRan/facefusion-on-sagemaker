## build image
在 facefusion-on-sagemaker 目录执行如下命令，如上docker file是已CPU举例的，可以修改使用GPU 可以参考gpu_Dockerfile
!./build_and_push.sh faces-swap-on-sagemaker

## local test
### 本地启动镜像
镜像构建完毕后，使用如下命令在Facefusion-Sagemaker-Studio-Lab目录执行,本地启动docker 镜像
./local_test/serv_local.sh facefusion-sagemaker

### 本地调用测试
执行 如下命令，测试本地endpoint 参数根据实际情况替换 

### 提交任务

curl -XPOST localhost:8080/invocations  -H 'content-type:application/json'  -d '{"input":"['-s','image1.jpg','-t','test.mp4','-o','/tmp/','-u','s3://sagemaker-us-west-2-687912291502/video/test_out.mp4','--headless']","method":"submit"}'

{"message": "Command executed in background"}

### 获取任务状态

curl -XPOST localhost:8080/invocations  -H 'content-type:application/json'  -d '{"input":"s3://zuimei-poc/facefusion/input/test_out.mp4","method":"get_status"}'

返回：{"status": "processing"}

## 构建sagemaker镜像
参考 
sagemaker-sync-endpoint.ipynb 创建


