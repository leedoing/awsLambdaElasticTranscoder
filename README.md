Using AWS Lambda, Elastic transcoder
====================================================

## Workflow
![alt tag](http://cfile21.uf.tistory.com/image/261EF445574F8CE30F1F8A)

## IAM Role
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

```
##Lambda(using a javscript sdk)
var aws = require('aws-sdk');
var elastictranscoder = new aws.ElasticTranscoder();
var s3 = new aws.S3();
var filename = function(path){ //upload object filename
    var pathSplit = path.split('.')[0];
    var arraySize = pathSplit.split('/').length-1;
    var filename = pathSplit.split('/')[arraySize];
    return filename;
}
var prefix = function(path){ //upload object prefix
    var pathSplit = path.split('.')[0];
    var arraySize = pathSplit.split('/').length-1;
    var prefix = '';
    for(var i=0; i<arraySize; i++){
        var text = pathSplit.split('/')[i];
        prefix=prefix.concat(text)+'/';
    }
    return prefix;
}
var listObject = function(params, callback){
    s3.listObjects(params, function(err, data){
        params.Delete = {Objects:[]};
        if(err) console.log(err, err.stack);
        else{
            console.log('listing Object...');
            data.Contents.forEach(function(result){
                params.Delete.Objects.push({Key: result.Key});
            })
            delete params.Prefix;
            callback(params);
            return
        }
    });
}
var deleteObject = function(params){
    listObject(params, function(deleteParams){
        console.log(deleteParams);
        s3.deleteObjects(deleteParams, function(err, data){
            if (err) console.log(err, err.stack); // an error occurreddeleteObjcects 
            else console.log(data);           // successful response
            console.log('delete Object...');
        });
    });
}
exports.handler = function(event, context) {
    console.log('Received event:', JSON.stringify(event, null, 2));
    // Get the object from the event and show its content type
    var key = event.Records[0].s3.object.key;
    var s3Params_5m = {
        Bucket: '5m_media-transcoding', //output bucket name
        Prefix: prefix(key) + filename(key) + '_5M.' + key.split('.')[1] + '/' //S3 In/Output Object Prefix
    };
    var s3Params_2m = {
        Bucket: '2m_media-transcoding', //output bucket name
        Prefix: prefix(key) + filename(key) + '_2M.' + key.split('.')[1] + '/'
    };
 
    deleteObject(s3Params_5m);
    deleteObject(s3Params_2m);
 
    var etsParams_5m = {
        Input: {
        Key: key
        },
        PipelineId: 'xxx-4o47oe', // pipeline ID
        OutputKeyPrefix: prefix(key) + filename(key) + '_5m.' + key.split('.')[1] + '/',
        Outputs: [
            {
                Key: filename(key),
                PresetId: 'xxxxxx-97mimb',
                SegmentDuration: '10'
            }
        ],
        Playlists: [
            {
                Format: 'HLSv3',
                Name: 'playlist',
                OutputKeys: [
                filename(key)
                ]
            }
        ]
    };
    var etsParams_2m = {
        Input: {
        Key: key
        },
        PipelineId: 'xxx-4o47oe', // pipeline ID
        OutputKeyPrefix: prefix(key) + filename(key) + '_2m.' + key.split('.')[1] + '/',
        Outputs: [
            {
                Key: filename(key),
                PresetId:'xxxxxx-5nj36s', //default copy
                SegmentDuration: '10'
            }
        ],
        Playlists: [
            {
                Format: 'HLSv3',
                Name: 'playlist',
                OutputKeys: [
                filename(key)
                ]
            }
        ]
    };
 
 
    elastictranscoder.createJob(etsParams_5m, function(err, data) {
        if(err){
            console.log('transcoding_5m failed');
            console.log(err, err.stack); // an error occurred
            context.fail();
        }else{
            console.log('transcoding_5m succeed')
            context.succeed('transcoding_5m succeed');
        }
    });
    elastictranscoder.createJob(etsParams_2m, function(err, data) {
        if(err){
            console.log('transcoding_2m failed');
            console.log(err, err.stack); // an error occurred
            context.fail();
        }else{
            console.log('transcoding_2m succeed')
            context.succeed('transcoding_2m succeed');
        }
    });
};
```
##AWS Document
[Configuring the SDK](http://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/ElasticTranscoder.html)

##Blog
I describe it on my blog in korean</br>
[leedoing](http://blog.leedoing.com/category/Application%20Service/ElasticTranscoder)

