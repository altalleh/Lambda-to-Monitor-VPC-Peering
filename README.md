# Lambda-to-Monitor-VPC-Peering
Using Lambda to get notification when more than 1 VPC exist in a certain region

Lmabda to monitor VPC Peering
To use Lambda sending notification when there is more than 1 VPC peering within a region using the API call DescribeVpcPeeringConnections, I created an example of a Lambda function and tested it as well, and I was able to receive emails when there is more than 1 VPC peering withing a region. 

To use and test my sample Lambda function, you can follow the below steps,  

[1] Permissions for your Lambda functions, where you need to allow the needed API calls like DescribeVpcPeeringConnections in addition to SNS as you need to public a topic via email. 

[2] Create a SNS Topic so you can use it to public notification from your Lambda function and subscribe your email to it.
- Create a Topic: http://docs.aws.amazon.com/sns/latest/dg/CreateTopic.html
- Create a subscription with your email address (you need to verify it from your email afterwards) : http://docs.aws.amazon.com/sns/latest/dg/SubscribeTopic.html

[3] Create your Lambda function. 

/*Start of code*/
console.log('Loading function');
var AWS = require('aws-sdk');

exports.handler = function(event, context) {
    var ec2 = new AWS.EC2({region: 'region where you want to check the VPC Peering for example us-east-1'})
    var params = {
    DryRun: false,
    };

    ec2.describeVpcPeeringConnections(params, function(err, data) {
    if (err) console.log(err, err.stack); // an error occurred
    else {
        console.log(data); // successful response
        console.log (Object.keys(data.VpcPeeringConnections).length); //Print how many VPC Peering you have in the region
        if (Object.keys(data.VpcPeeringConnections).length>0){
            var sns = new AWS.SNS({region: 'region where you created the sns topic'});
            var params = {
              Message: 'You have more than 0 VPC Peering in EU-West-1', /* required */
              MessageStructure: 'Raw',
              Subject: 'VPC Peering Count Notification',
              TopicArn: 'arn:aws:sns:the_rest_of_your_sns_arn'
            };
            sns.publish(params, function(err, data) {
              if (err) console.log(err, err.stack); // an error occurred
              else     console.log(data);           // successful response
            });
            }
            else {console.log("Not sending email cause there is now")}
    }
    
    });
};
/*end of code*/

[4] Create an event schedule for your Lambda function by going to Event sources -> Add event source -> then set the parameters,
Event source type: CloudWatch Events - Schedule
Rule name : //give it a name
Rule description: //give it a description
Schedule expression: //choose the right rate that you prefer

This should help you send email notification when you have more than 1 VPC peering within a region. 
