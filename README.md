Using AWS Lambda, Elastic transcoder
====================================================

>## Using a S3, IAM, Lambda and Elastic Transcoder
>## IAM Role
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:*:*:*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:Put*",
                "s3:Get*",
                "s3:*MultipartUpload*",
                "s3:Delete*"
            ],
            "Resource": [
                "arn:aws:s3:::*"
            ]
        },
        {
            "Sid": "Stmt1441234334958",
            "Action": [
                "elastictranscoder:CreateJob"
            ],
            "Effect": "Allow",
            "Resource": "*"
        }
    ]
}
```
>##Lambda(using a javscript sdk)
```javascript
var aws = require('aws-sdk');
var elastictranscoder = new aws.ElasticTranscoder();
var s3 = new aws.S3();
exports.handler = function(event, context) {
    console.log('Received event:', JSON.stringify(event, null, 2));
    // Get the object from the event and show its content type
    var key = event.Records[0].s3.object.key;
    var s3Params = {
        Bucket: 'leedoing-transcoding', //output bucket name
        Key: key.split('.')[0]+'.mp4' //object name
    };
    var etParams = {
      Input: {
        Key: key
      },
      PipelineId: '1464587330316-a37yxe', // pipeline ID
      Outputs: [
        {
            Key: key.split('.')[0]+'.mp4',
            PresetId: '1351620000001-000001', //transcoding presetID(ex. gerneric1080p)
            Watermarks: [
                {
                InputKey: 'watermark.png',
                PresetWatermarkId: 'TopRight'
                }
            ]
        }
      ]
    };
    s3.deleteObject(s3Params, function(err, data){ //async(deleteobject, transcoding)
        if(err){
            console.log('s3 object delete failed');
            console.log(err, err.stack);
            context.fail();
        }else{
            console.log('s3 objcet delete succceed');
            elastictranscoder.createJob(etParams, function(err, data) {
                if(err){
                    console.log('transcoding failed');
                    console.log(err, err.stack); // an error occurred
                    context.fail();
                }else{
                    console.log('transcoding succeed')
                    context.succeed('transcoding succeed');
                }
            });
        } 
    });
};
```
>##AWS Document
[Configuring the SDK](http://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/ElasticTranscoder.html)

>##Blog
>I explained this workflow </br>
>[leedoing](http://blog.leedoing.com/category/Application%20Service/ElasticTranscoder)

