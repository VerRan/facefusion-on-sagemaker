FROM nvidia/cuda:11.8.0-cudnn8-runtime-ubuntu22.04

ARG DEBIAN_FRONTEND=noninteractive
ARG FACEFUSION_VERSION=2.3.0
ENV GRADIO_SERVER_NAME=0.0.0.0
ENV PYTHONUNBUFFERED=TRUE
ENV PYTHONDONTWRITEBYTECODE=TRUE
ENV PATH="/opt/program:${PATH}"

WORKDIR /opt/program

RUN apt-get update
RUN apt-get install python3.10 -y
RUN apt-get install python-is-python3 -y
RUN apt-get install pip -y
RUN apt-get install git -y
RUN apt-get install curl -y
RUN apt-get install ffmpeg -y


##安装sagemaker endpoint所需的组件
RUN apt-get install nginx -y  
RUN pip install --no-cache-dir boto3 flask gunicorn
# RUN git clone https://github.com/facefusion/facefusion.git --branch ${FACEFUSION_VERSION} --single-branch .
##拷贝包含sagemaker endpoint所需的python和配置文件
COPY facefusion /opt/program
RUN python install.py --torch cuda --onnxruntime cuda
RUN cd /usr/local/lib/python3.10/dist-packages/torch/lib && ln -s libnvrtc-672ee683.so.11.2 libnvrtc.so

WORKDIR /opt/program

# Expose port 8080 for serving
EXPOSE 8080

ENTRYPOINT ["python"]

# serve is a python script under code/ directory that launches nginx and gunicorn processes
CMD [ "serve" ]