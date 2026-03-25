# AWS S3 to Snowflake Data Loading

## Objective
Load data from AWS S3 into Snowflake using:
- Storage Integration
- External Stage
- Snowpipe

## Architecture
AWS S3 → IAM Role → Storage Integration → External Stage → Snowpipe → Snowflake Table

## Step 1: AWS Setup

Create:
- AWS Free Tier Account
- S3 Bucket
- IAM Role

Bucket Name:
data-loading-snowflake-aws 

Folder:
Hotel_Booking

Upload dataset files inside the Hotel_Booking folder.

---

### Create IAM Role

Follow the steps below to create an IAM Role:

1. Go to **AWS Console**
2. Navigate to **IAM**
3. Click **Roles**
4. Click **Create Role**

---

### Select Trusted Entity

- Choose **AWS Account**
- Click **Next**

---

### Add Permissions

Attach permission:

- `AmazonS3FullAccess`

Click **Next**

---

### Role Details

Enter the following:

- Role Name: `snowflake-s3-role`
- Description: `Role for Snowflake to access S3 bucket`

Click **Create Role**

Copy the following values:

1. STORAGE_AWS_IAM_USER_ARN
2. STORAGE_AWS_EXTERNAL_ID

These values are required to update AWS IAM Trust Relationship

## Create Storage Integration


```
CREATE OR REPLACE STORAGE INTEGRATION STG
TYPE = EXTERNAL_STAGE
ENABLED = TRUE
STORAGE_PROVIDER = 'S3'
STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::975050328425:role/snowflake-s3-role'
STORAGE_ALLOWED_LOCATIONS = ('s3://data-loading-snowflake-aws/Hotel_Booking/');
```

## Get Snowflake IAM User

Then copy:

STORAGE_AWS_IAM_USER_ARN

Use this values to update AWS IAM Trust Relationship


```
DESC STORAGE INTEGRATION STG;
```

### Edit Trust Policy

 Update the **Trust Policy** to allow Snowflake to assume the role.

Follow the steps below:

1. Go to **AWS Console**
2. Navigate to **IAM**
3. Click **Roles**
4. Select the created role (`snowflake-s3-role`)
5. Go to **Trust Relationships**
6. Click **Edit Trust Policy**

### Update Trust Policy

Update the existing trust policy by replacing the **Principal AWS value** with  
`STORAGE_AWS_IAM_USER_ARN` obtained from Snowflake.

## Create External Stage


```
CREATE OR REPLACE STAGE AWS_STAGE
STORAGE_INTEGRATION = STG
URL = 's3://data-loading-snowflake-aws/Hotel_Booking/';
```


```
LIST @AWS_STAGE;
```


```
Select $1,$2 from @AWS_STAGE;
```

## Create Table


```
CREATE OR REPLACE TABLE HOTEL_BOOKINGS (
    booking_id VARCHAR,
    hotel_id VARCHAR,
    hotel_city VARCHAR,
    customer_id VARCHAR,
    customer_name VARCHAR,
    customer_email VARCHAR,
    check_in_date DATE,
    check_out_date DATE,
    room_type VARCHAR,
    num_guests INTEGER,
    total_amount FLOAT,
    currency VARCHAR,
    booking_status VARCHAR
);
```

## Load Data


```
COPY INTO HOTEL_BOOKINGS
FROM @AWS_STAGE
FILE_FORMAT = (TYPE = CSV SKIP_HEADER = 1)
ON_ERROR = 'CONTINUE';
```

## Validate


```
SELECT * FROM HOTEL_BOOKINGS LIMIT 10;
```
