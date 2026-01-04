# AWS Glue 5.0 Local Development Environment

This guide will help you set up and run AWS Glue 5.0 locally using Docker. This setup includes JupyterLab with SparkMagic and Livy server for interactive Spark development.

## 📋 Prerequisites

- Docker installed and running
- Docker Hub account (for pulling images)
- AWS credentials configured (for AWS service access)
- Bash or Zsh shell

## 🚀 Quick Start

### Step 1: Pull the AWS Glue 5.0 Base Image

First, logout and pull the official AWS Glue 5.0 image:

```bash
docker logout public.ecr.aws
docker pull public.ecr.aws/glue/aws-glue-libs:5
```

### Step 2: Set Up Your Workspace

Configure your workspace location and AWS profile:

```bash
export WORKSPACE_LOCATION=/Users/soumilshah/IdeaProjects/icebergpython/glue5-locally/workspace
export PROFILE_NAME="default"  # Change to your AWS profile name if different
mkdir -p $WORKSPACE_LOCATION/jupyterlab/
```

**Note:** Update `WORKSPACE_LOCATION` to match your project directory path.

### Step 3: Build the Docker Image

Navigate to the jupyterlab directory and build the custom Glue 5.0 image with JupyterLab and Livy:

```bash
cd $WORKSPACE_LOCATION/jupyterlab/
docker build \
    -t glue_v5_livy \
    --file $WORKSPACE_LOCATION/jupyterlab/Dockerfile.livy_jupyter \
    $WORKSPACE_LOCATION/jupyterlab/
```

This build process will:
- Use the official AWS Glue 5.0 base image
- Install Livy server (Apache Livy 0.8.0)
- Install JupyterLab with SparkMagic
- Configure Spark kernels for PySpark and Spark

### Step 4: Run the Container

Start the Glue 5.0 container with volume mounts and port mappings:

```bash
docker run -d --rm \
    -v ~/.aws:/home/hadoop/.aws \
    -v $WORKSPACE_LOCATION:/home/hadoop/workspace/ \
    -p 8998:8998 \
    -p 8888:8888 \
    -e AWS_PROFILE=$PROFILE_NAME \
    --name glue5_jupyter \
    glue_v5_livy
```

**Ports:**
- `8888`: JupyterLab web interface
- `8998`: Livy server REST API

**Volumes:**
- `~/.aws`: AWS credentials directory (read-only mount)
- `$WORKSPACE_LOCATION`: Your workspace directory (read-write mount)

### Step 5: Access JupyterLab

Open your browser and navigate to:

```
http://localhost:8888
```

You should see the JupyterLab interface. The workspace directory will be mounted at `/home/hadoop/workspace/`, so your notebooks will persist in your local filesystem.

## 📓 Using Jupyter Notebooks

Once in JupyterLab:

1. Create a new notebook
2. Select either the **PySpark** or **Spark** kernel
3. Start writing and executing your Spark/Glue code

Example PySpark code:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("Glue5Example") \
    .getOrCreate()

df = spark.range(10)
df.show()
```

## 💻 Command Line Usage

### Start PySpark Shell

Access an interactive PySpark shell inside the running container:

```bash
docker exec -it glue5_jupyter pyspark
```

### Submit Spark Jobs

Run Spark jobs using `spark-submit`:

```bash
docker exec -it glue5_jupyter \
  spark-submit \
  --class org.apache.spark.examples.SparkPi \
  /usr/lib/spark/examples/jars/spark-examples_2.12-3.5.4-amzn-0.jar \
  100
```

Replace the class and jar path with your own Spark application.

## 🛠️ Container Management

### View Container Logs

```bash
docker logs glue5_jupyter
```

### Stop the Container

```bash
docker stop glue5_jupyter
```

Since the container was started with `--rm`, it will be automatically removed when stopped.

### Restart the Container

```bash
docker start glue5_jupyter
```

### Access Container Shell

```bash
docker exec -it glue5_jupyter bash
```

## 📁 Project Structure

```
glue5-locally/
├── README.md
├── notes
└── workspace/
    ├── demo.ipynb
    └── jupyterlab/
        └── Dockerfile.livy_jupyter
```

## 🔧 Customization

### Change AWS Profile

Update the `PROFILE_NAME` environment variable before running the container:

```bash
export PROFILE_NAME="my-aws-profile"
```

### Change Workspace Location

Modify the `WORKSPACE_LOCATION` variable to point to your desired directory:

```bash
export WORKSPACE_LOCATION="/path/to/your/workspace"
```

### Modify Ports

If ports 8888 or 8998 are already in use, map them to different host ports:

```bash
docker run -d --rm \
    -v ~/.aws:/home/hadoop/.aws \
    -v $WORKSPACE_LOCATION:/home/hadoop/workspace/ \
    -p 8889:8888 \  # Map host port 8889 to container port 8888
    -p 8999:8998 \  # Map host port 8999 to container port 8998
    -e AWS_PROFILE=$PROFILE_NAME \
    --name glue5_jupyter \
    glue_v5_livy
```

## 🐛 Troubleshooting

### Container Won't Start

1. Check if ports are already in use:
   ```bash
   lsof -i :8888
   lsof -i :8998
   ```

2. View container logs:
   ```bash
   docker logs glue5_jupyter
   ```

### AWS Credentials Not Working

Ensure your AWS credentials are properly configured:

```bash
cat ~/.aws/credentials
```

Verify the AWS profile matches the `PROFILE_NAME` environment variable.

### JupyterLab Not Accessible

1. Verify the container is running:
   ```bash
   docker ps | grep glue5_jupyter
   ```

2. Check if the port is correctly mapped:
   ```bash
   docker port glue5_jupyter
   ```

### Build Fails

If the Docker build fails:

1. Ensure you have a stable internet connection
2. Verify Docker has enough resources allocated
3. Check Docker logs during build:
   ```bash
   docker build --progress=plain -t glue_v5_livy --file $WORKSPACE_LOCATION/jupyterlab/Dockerfile.livy_jupyter $WORKSPACE_LOCATION/jupyterlab/
   ```

## 📚 Additional Resources

- [AWS Glue Documentation](https://docs.aws.amazon.com/glue/)
- [Apache Spark Documentation](https://spark.apache.org/docs/latest/)
- [Apache Livy Documentation](https://livy.apache.org/)
- [SparkMagic Documentation](https://github.com/jupyter-incubator/sparkmagic)

## 📝 Notes

- The workspace directory is mounted as a volume, so all your notebooks and files will persist locally
- AWS credentials are mounted read-only from `~/.aws`
- The container uses the `hadoop` user inside the container
- JupyterLab is configured without password/token for local development (not recommended for production)

---

**Happy Glue Development! 🎉**

