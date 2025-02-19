I'll create a requirements.txt file and update the README with detailed instructions.

```text:requirements.txt
pyspark>=3.2.0
yfinance>=0.2.0
redis>=4.5.0
pandas>=1.5.0
boto3>=1.26.0
```

```markdown:README.md
# Stock Data Pipeline

A scalable data pipeline for processing OHLCV (Open, High, Low, Close, Volume) data for stocks, forex, and futures using PySpark and Redis.

## Features

- Batch processing with PySpark
- Real-time data access via Redis cache
- Technical indicators calculation (SMA, RSI, Volatility)
- S3 storage with Parquet format
- Configurable for multiple time intervals (daily, hourly, 15-min)

## Local Setup

### Prerequisites

- Python 3.8+
- Java 8+
- Redis server
- AWS credentials configured (for S3 access)

### Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd stock-data-pipeline
```

2. Create and activate a virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install dependencies:
```bash
pip install -r requirements.txt
```

4. Configure Redis:
- Install Redis server (if not already installed)
  - Ubuntu: `sudo apt-get install redis-server`
  - macOS: `brew install redis`
  - Windows: Download from [Redis Windows](https://github.com/microsoftarchive/redis/releases)
- Start Redis server:
  - Linux/macOS: `redis-server`
  - Windows: Start Redis service

5. Update configuration:
- Edit `config.py` to set your S3 bucket and Redis configuration
- Ensure AWS credentials are configured (`~/.aws/credentials` or environment variables)

### Running Locally

1. Start the pipeline:
```bash
python pipeline.py
```

## AWS EMR Deployment

### Prerequisites

- AWS CLI installed and configured
- S3 bucket created for code and data
- EMR IAM roles configured

### Deployment Steps

1. Package the application:
```bash
zip -r stock_pipeline.zip *.py requirements.txt
```

2. Upload to S3:
```bash
aws s3 cp stock_pipeline.zip s3://your-bucket/code/stock_pipeline.zip
```

3. Create EMR cluster:
```bash
aws emr create-cluster \
    --name "Stock Pipeline Cluster" \
    --release-label emr-6.9.0 \
    --applications Name=Spark Name=Python \
    --ec2-attributes KeyName=your-key-pair \
    --instance-type m5.xlarge \
    --instance-count 3 \
    --bootstrap-actions Path="s3://your-bucket/code/bootstrap.sh" \
    --use-default-roles
```

4. Create bootstrap script (`bootstrap.sh`):
```bash
#!/bin/bash
sudo yum install -y redis
sudo service redis start
sudo pip3 install -r /home/hadoop/requirements.txt
```

5. Submit the job:
```bash
aws emr add-steps \
    --cluster-id j-XXXXXXXXXXXXX \
    --steps Type=Spark,Name="Stock Pipeline",ActionOnFailure=CONTINUE,\
    Args=[--deploy-mode,cluster,--master,yarn,--py-files,s3://your-bucket/code/stock_pipeline.zip,s3://your-bucket/code/pipeline.py]
```

### Scheduling

To run the pipeline daily:

1. Using AWS EventBridge (CloudWatch Events):
```bash
aws events put-rule \
    --name "DailyStockPipeline" \
    --schedule-expression "cron(0 0 * * ? *)"
```

2. Create a target for the rule that triggers an EMR step.

### Monitoring

- Monitor EMR cluster health in AWS Console
- Check EMR logs in CloudWatch
- View job progress in Spark UI
- Monitor Redis cache using Redis CLI:
```bash
redis-cli monitor
```

## Architecture

```
┌─────────────┐    ┌──────────┐    ┌────────────┐
│ Data Source │ -> │ PySpark  │ -> │ Feature    │
│ (YFinance)  │    │ Process  │    │ Engineering│
└─────────────┘    └──────────┘    └────────────┘
                         │                │
                         v                v
                   ┌──────────┐    ┌────────────┐
                   │ S3       │    │ Redis      │
                   │ Storage  │    │ Cache      │
                   └──────────┘    └────────────┘
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Commit changes
4. Push to the branch
5. Create a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.
```

This README provides comprehensive instructions for both local development and AWS EMR deployment. The instructions include all necessary steps for setting up the environment, running the pipeline, and monitoring the deployment. The architecture diagram helps visualize the data flow, and the contributing section encourages collaboration.
