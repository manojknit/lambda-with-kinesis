 # Big Data Processing with Amazon Kinesis and lambda

1. Go To AWS IAM. Create an execution role and assign Permissions – AWSLambdaKinesisExecutionRole.
2. Create lambda with following file index.js
```
console.log('Loading function');
exports.handler = function(event, context) {
    //console.log(JSON.stringify(event, null, 2));
    event.Records.forEach(function(record) {
        // Kinesis data is base64 encoded so decode here
        var payload = new Buffer(record.kinesis.data, 'base64').toString('ascii');
        console.log('Decoded payload:', payload);
    });
};
```
3. Zip as deployment package
```
$ zip lambdawithkinesisfunction.zip index.js
```

4. Deploy lambda 
```
aws lambda create-function --function-name lambdawithkinesisfunction \
--zip-file fileb://lambdawithkinesisfunction.zip --handler index.handler --runtime nodejs8.10 \
--role arn:aws:iam::494875521123:role/service-role/iPromoLambdaRole
```
```
C02TN40KHTD5:lambda-with-kinesis mk194903$ aws lambda create-function --function-name lambdawithkinesisfunction \
> --zip-file fileb://lambdawithkinesisfunction.zip --handler index.handler --runtime nodejs8.10 \
> --role arn:aws:iam::494875521123:role/service-role/iPromoLambdaRole
{
    "TracingConfig": {
        "Mode": "PassThrough"
    },
    "CodeSha256": "QDCBfPGq8k2+aOBZgjOtSO8WbwihfKC4ibm2Sktf6UI=",
    "FunctionName": "lambdawithkinesisfunction",
    "CodeSize": 394,
    "RevisionId": "ccfede5e-13bb-4e53-a618-45d42d71febd",
    "MemorySize": 128,
    "FunctionArn": "arn:aws:lambda:us-east-1:494875521123:function:lambdawithkinesisfunction",
    "Version": "$LATEST",
    "Role": "arn:aws:iam::494875521123:role/service-role/iPromoLambdaRole",
    "Timeout": 3,
    "LastModified": "2019-05-26T21:20:21.614+0000",
    "Handler": "index.handler",
    "Runtime": "nodejs8.10",
    "Description": ""
}
```
5. Invoke to run lambda
```
$ aws lambda invoke --function-name lambdawithkinesisfunction --payload file://input.txt out.txt
 Response is saved to out.txt
```
 6. Create Kinesis Stream
```
$ aws kinesis create-stream --stream-name lambdawithkinesis-stream --shard-count 1
```
 
 7. Run descrive to get new created kinesis. 
 ```
 $ aws kinesis describe-stream --stream-name lambdawithkinesis-stream
 ```
 ```
 C02TN40KHTD5:lambda-with-kinesis mk194903$ aws kinesis describe-stream --stream-name lambdawithkinesis-stream
{
    "StreamDescription": {
        "KeyId": null,
        "EncryptionType": "NONE",
        "StreamStatus": "ACTIVE",
        "StreamName": "lambdawithkinesis-stream",
        "Shards": [
            {
                "ShardId": "shardId-000000000000",
                "HashKeyRange": {
                    "EndingHashKey": "340282366920938463463374607431768211455",
                    "StartingHashKey": "0"
                },
                "SequenceNumberRange": {
                    "StartingSequenceNumber": "49596093639113802764351019997592281861991808571392655362"
                }
            }
        ],
        "StreamARN": "arn:aws:kinesis:us-east-1:494875521123:stream/lambdawithkinesis-stream",
        "EnhancedMonitoring": [
            {
                "ShardLevelMetrics": []
            }
        ],
        "StreamCreationTimestamp": 1558906237.0,
        "RetentionPeriodHours": 24
    }
}
```
8. Add the event source to lambda
```
$ aws lambda create-event-source-mapping --function-name lambdawithkinesisfunction \
--event-source  arn:aws:kinesis:us-east-1:494875521123:stream/lambdawithkinesis-stream \
--batch-size 100 --starting-position LATEST
```
9. Running the list-event-source-mappings command to verify event mapping.
```
$ aws lambda list-event-source-mappings --function-name lambdawithkinesisfunction \
--event-source arn:aws:kinesis:us-east-1:494875521123:stream/lambdawithkinesis-stream
```
```
C02TN40KHTD5:lambda-with-kinesis mk194903$ aws lambda list-event-source-mappings --function-name lambdawithkinesisfunction \
> --event-source arn:aws:kinesis:us-east-1:494875521123:stream/lambdawithkinesis-stream
{
    "EventSourceMappings": [
        {
            "UUID": "73f3f279-edb4-4120-a9bb-a336790294cc",
            "StateTransitionReason": "User action",
            "LastModified": 1558906800.0,
            "BatchSize": 100,
            "State": "Enabled",
            "FunctionArn": "arn:aws:lambda:us-east-1:494875521123:function:lambdawithkinesisfunction",
            "EventSourceArn": "arn:aws:kinesis:us-east-1:494875521123:stream/lambdawithkinesis-stream",
            "LastProcessingResult": "No records processed"
        }
    ]
}
```
10. Let's test the stream
```
$ aws kinesis put-record --stream-name lambdawithkinesis-stream --partition-key 1 \
--data "Hi, this is my first test."
```
This message can be verified in cloudwatch.

#### Reference:
[Serverless Data Processing](https://aws.amazon.com/lambda/data-processing/?trk=ps_a131L000005OoXQQA0)
[How we built a data pipeline with Lambda Architecture using Spark/Spark Streaming](https://medium.com/walmartlabs/how-we-built-a-data-pipeline-with-lambda-architecture-using-spark-spark-streaming-9d3b4b4555d3)
