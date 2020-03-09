# DynamoDB Global Tables Demostration

## Step 1

Create a new DynamoDB table named ```cloudacademy-courses```.

```
aws dynamodb create-table \
    --region us-west-2 \
    --table-name cloudacademy-courses \
    --key-schema AttributeName=courseid,KeyType=HASH \
    --attribute-definitions AttributeName=courseid,AttributeType=S \
    --billing-mode PAY_PER_REQUEST
```

## Step 2

Load course data into the ```cloudacademy-courses``` table.

```
aws dynamodb batch-write-item --region us-west-2 --request-items file://./batch.course.data1.json
```

## Step 3

Update the ```cloudacademy-courses``` table and make it a global table with a new replica in the ```ap-southeast-2``` (Sydney) region.

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

Check to see if the ```cloudacademy-courses``` global table is ready in the ```ap-southeast-2``` region.

**Note**: Wait 1-2 minutes before running this command to give the previous command enough time to propagate the table updates across to the new region.

```
aws dynamodb describe-table \
    --region ap-southeast-2 \
    --table-name cloudacademy-courses
```

## Step 5

Setup 30 second watch on the ```cloudacademy-courses``` global table deployed in the ```ap-southeast-2``` region - and query the ```TableStatus``` attribute. Wait for it to reach ```Active``` status.

**Note**: This command leverages the [jq utility](https://stedolan.github.io/jq/) to query the json data response for the ```TableStatus``` attribute

```
watch -n 30 "aws dynamodb describe-table \
    --region ap-southeast-2 \
    --table-name cloudacademy-courses \
    | jq -r .Table.TableStatus"
```

## Step 6

Start up ```tmux``` and split the terminal. 

**Pane 1**: Run the following command to query for a new record which we will load in the second tmux pane.

```
watch -n1 "aws dynamodb get-item --region ap-southeast-2 --table-name cloudacademy-courses --key '{\"courseid\" : {\"S\" : \"6666666\"}}'"
```

**Pane 2**: Load the new data set into the ```cloudacademy-courses```. Observe how long it takes before the results show up in the other tmux pane.

```
aws dynamodb batch-write-item --region us-west-2 --request-items file://./batch.course.data2.json
```