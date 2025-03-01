# Weather ETL Pipeline Using Apache Airflow

## Project Overview

This project demonstrates how to build and automate an ETL process using Apache Airflow. The pipeline extracts current weather data from the OpenWeatherMap API, transforms it, and loads it into an AWS S3 bucket.

The project is entirely carried out on AWS EC2 for running Airflow and an S3 bucket for data storage. Key Airflow concepts such as DAGs, Operators, and Sensors are covered.

---

## Architecture

<img width="674" alt="Image" src="https://github.com/user-attachments/assets/2b23cd18-7329-45fd-baf8-e4eba761dfac" />


---

## Resources
 
- **Weather API**: [OpenWeatherMap API](https://openweathermap.org/api)   

## Setup Instructions

### 1. Create an AWS EC2 Instance
- Launch an EC2 instance (recommended: `t2.small` type)
- Create and download a new key pair (`.pem` file)
- Edit inbound rules to allow TCP 8080 (required for Airflow UI access)

### 2. Install Dependencies on EC2
Run the following commands:
```sh
sudo apt update
sudo apt install python3-pip
sudo apt install python3.10-venv
python3 -m venv airflow_venv
source airflow_venv/bin/activate
pip install pandas s3fs apache-airflow
```

## 3. Connect to OpenWeatherMap API

1. Sign up at OpenWeatherMap and generate an API key.

2. **Add API Key to Airflow:**
   - In the Airflow UI, navigate to **Admin > Connections**.
   - Create a new connection:

     ```
     Conn ID: weathermap_api
     Conn Type: HTTP
     Host: https://api.openweathermap.org
     ```

   - Extra (JSON format):

     ```json
     { "apikey": "your_api_key_here" }
     ```

## 4. Configure Airflow

- Airflow generates a username and password upon installation.
- Copy the **public IPv4 DNS** of the EC2 instance and access the Airflow UI at:


- Add AWS credentials in Python code for storing CSV files in S3.

     ```
     http://<public-ipv4>:8080
     ```
  
Log in using `admin` as the username and the generated password.

---

## 5. Connect VS Code to EC2 (Optional)

- Open VS Code

- Inside SSH configuration (`~/.ssh/config`), add a new host:

    ```
    Host ec2-instance
    Hostname <public-ip>
    User ubuntu
    IdentityFile ~/.ssh/<your-key>.pem
    ```


- Connect to EC2 and navigate to `/home/ubuntu` to manage Airflow files.

---

## Project Code

## `weather_dag.py`

The DAG performs the following tasks:

- **Check if the Weather API is available** using an `HttpSensor`
- **Extract weather data** using `SimpleHttpOperator`
- **Transform and Load data into S3** using `PythonOperator`

### Data Transformation Improvements

- Converted temperature from Kelvin to Fahrenheit for better readability.
- Adjusted timestamps to local timezones for accurate record-keeping.

---

## Running the DAG

Run the following command to start Airflow:

```
airflow standalone
```

Open the **DAGs** tab in Airflow UI and trigger the DAG manually.

Monitor task execution through **Graph View** and **Logs**.

---

## Storing Data in AWS S3

### Create an S3 Bucket

- Grant EC2 access to S3 by creating an **IAM role with S3 permissions**.

### Install AWS CLI and configure credentials:

```
sudo apt install awscli
aws configure
```

- Add AWS credentials in Python code for storing CSV files in S3. 


---

## Challenges and Solutions

### 1. API Rate Limits

- **Issue**: OpenWeatherMap API imposes rate limits, causing occasional failures.
- **Solution**: Implemented a retry mechanism in Airflow to handle failures gracefully.

### 2. Timezone Handling

- **Issue**: The API returns timestamps in UTC, which was not intuitive for data analysis.
- **Solution**: Adjusted timestamps using Python's `datetime` module to reflect local time zones.

### 3. AWS Permissions Issues

- **Issue**: Initially, the EC2 instance lacked permissions to write to S3.
- **Solution**: Created an IAM role with S3 access and attached it to the EC2 instance.

---

## Output

- A **CSV file** with transformed weather data is generated and uploaded to **S3**.
- The **ETL pipeline** automates data extraction, transformation, and storage.

---

## Future Enhancements

- Implement error handling for API failures.
- Add notifications for DAG execution status.
- Store data in a database for better analytics.

