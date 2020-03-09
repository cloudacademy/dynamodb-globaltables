# dynamodb-globaltables

## Step 1

```
aws dynamodb create-table \
    --region us-west-2 \
    --table-name cloudacademy-courses \
    --key-schema AttributeName=courseid,KeyType=HASH \
    --attribute-definitions AttributeName=courseid,AttributeType=S \
    --billing-mode PAY_PER_REQUEST
```

## Step 2

```
aws dynamodb batch-write-item --region us-west-2 --request-items file://./batch.course.data1.json
```

## Step 3

```
aws dynamodb update-table \
    --region us-west-2 \
    --table-name cloudacademy-courses --cli-input-json  \
'{
  "ReplicaUpdates":
  [
    {
      "Create": {
        "RegionName": "ap-southeast-2"
      }
    }
  ]
}'
```

## Step 4

```
aws dynamodb describe-table \
    --region us-west-2 \
    --table-name cloudacademy-courses
```

## Step 5

```
watch -n 30 "aws dynamodb describe-table \
    --region ap-southeast-2 \
    --table-name cloudacademy-courses \
    | jq -r .Table.TableStatus"
```

## Step 6

```
watch -n1 "aws dynamodb get-item --region ap-southeast-2 --table-name cloudacademy-courses --key '{\"courseid\" : {\"S\" : \"6666666\"}}'"
```

## Step 7

```
aws dynamodb batch-write-item --region us-west-2 --request-items file://./batch.course.data2.json
```