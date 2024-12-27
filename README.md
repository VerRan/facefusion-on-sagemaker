# FaceFusion on SageMaker

This project provides deployment configurations and scripts for running FaceFusion on Amazon SageMaker, supporting both CPU and GPU environments with synchronous and asynchronous inference capabilities.

## Project Structure

```
.
├── async_inference.ipynb      # Notebook for asynchronous inference setup
├── build_and_push.sh         # Script to build and push Docker image to ECR
├── cpu_Dockerfile            # Dockerfile for CPU-based deployment
├── Dockerfile               # Default Dockerfile
├── gpu_Dockerfile           # Dockerfile for GPU-based deployment
├── sagemaker_sync_endpoint.ipynb  # Notebook for synchronous endpoint setup
├── start_endpoint.sh        # Script to start the endpoint
├── start.sh                # General startup script
└── facefusion/             # FaceFusion model files
```

## Prerequisites

- AWS Account with SageMaker access
- Docker installed locally
- AWS CLI configured
- Python 3.10+
- Required Python packages:
  - boto3
  - sagemaker
  - sagemaker_ssh_helper (for debugging)

## Setup Options

### 1. CPU Deployment

Uses the CPU-optimized Dockerfile for environments without GPU requirements:

```bash
# Build and push the CPU image
./build_and_push.sh faces-swap-on-sagemaker
```

### 2. GPU Deployment

For enhanced performance using GPU acceleration:

1. Use the `gpu_Dockerfile` which is based on NVIDIA CUDA 11.8
2. Build and push using:
```bash
# Build and push the GPU image
./build_and_push.sh faces-swap-on-sagemaker gpu
```

## Local Testing

1. Start the local Docker container:
```bash
./local_test/serv_local.sh facefusion-sagemaker
```

2. Test the endpoint:
```bash
# Submit a task
curl -XPOST localhost:8080/invocations -H 'content-type:application/json' \
-d '{"input":"['-s','image1.jpg','-t','test.mp4','-o','/tmp/','-u','s3://your-bucket/output/test_out.mp4','--headless']","method":"submit"}'

# Check task status
curl -XPOST localhost:8080/invocations -H 'content-type:application/json' \
-d '{"input":"s3://your-bucket/output/test_out.mp4","method":"get_status"}'
```

## SageMaker Deployment

### Synchronous Endpoint

Use `sagemaker_sync_endpoint.ipynb` to:
1. Create a SageMaker model
2. Configure the endpoint
3. Deploy the endpoint
4. Test real-time inference

### Asynchronous Endpoint

Use `async_inference.ipynb` to:
1. Set up asynchronous inference configuration
2. Deploy with auto-scaling capabilities
3. Handle long-running inference tasks
4. Process larger payloads (up to 1GB)

## API Usage

### Synchronous Endpoint

```python
import boto3

runtime_sm_client = boto3.client("sagemaker-runtime")
response = runtime_sm_client.invoke_endpoint(
    EndpointName="your-endpoint-name",
    ContentType="application/json",
    Body=json.dumps({
        "method": "submit",
        "input": ['-s','image1.jpg','-t','test.mp4','-o','/tmp/',
                 '-u','s3://your-bucket/output/test_out.mp4','--headless']
    })
)
```

### Asynchronous Endpoint

```python
response = runtime_sm_client.invoke_endpoint_async(
    EndpointName="your-endpoint-name",
    InputLocation="s3://input-bucket/input.json"
)
output_location = response["OutputLocation"]
```

## Infrastructure as Code

The project includes AWS CDK templates for automated deployment:
- `SageMakerStack`: Creates the complete infrastructure
- Supports both serverless and instance-based endpoints
- Configurable auto-scaling policies

## Environment Variables

- `GRADIO_SERVER_NAME`: Set to 0.0.0.0 for container access
- `PYTHONUNBUFFERED`: Ensures immediate log output
- `PYTHONDONTWRITEBYTECODE`: Prevents .pyc file generation

## Security Considerations

- Ensure proper IAM roles and permissions
- Configure VPC endpoints if needed
- Use encryption for data in transit and at rest
- Implement appropriate security groups

## Monitoring and Maintenance

- CloudWatch integration for logs and metrics
- Auto-scaling policies for cost optimization
- Endpoint status monitoring
- Regular updates and maintenance

## License

Include your license information here.
