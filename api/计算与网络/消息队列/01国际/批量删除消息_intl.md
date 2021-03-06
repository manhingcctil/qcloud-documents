## 1. API Description

This API (BatchDeleteMessage) is used to delete a batch of messages (currently you can delete up to 16 messages at a time) that have already been consumed. The consumer needs to use the ReceiptHandle (obtained after the last consume operation) as parameter to locate the messages to be deleted. You can only execute this operation before NextVisibleTime. Messages will return to Active status after NextVisibleTime and ReceiptHandle will become invalid, causing deletion operation to fail. In this case, you will need to re-consume the messages and acquire ReceiptHandle again. Under concurrent consuming scenario, if a message is deleted by one of the consumers, the other consumers will no longer be able to obtain the deleted message.

Domain for public network API request:<font style="color:red">cmq-queue-region.api.qcloud.com</font>

Domain for private network API request:<font style="color:red">cmq-queue-region.api.tencentyun.com</font>

> At any time (including alpha test), any downstream traffic generated when using public network domain will incur traffic fee. It is strongly recommended that users on Tencent Cloud use **private network** domain, as it will not incur any traffic fee.

- The "region" should be replaced by a specific region: gz (Guangzhou), sh (Shanghai), bj (Beijing). The "region" value in the common parameter should be kept consistent with that in the domain. In case of inconsistency, the one in the domain should prevail. The request should be sent to the region specified by the domain.
- Requests for accessing via public network domain support both HTTP and HTTPS. Requests for accessing via private network only support HTTP.
- Some of the input parameters are optional, so the default values are not required.
- All output parameters will be returned to the user when the request is successful; otherwise, at least code, message, and requestId will be returned.


## 2. Input Parameters

The following request parameter list only provides API request parameters. For other parameters, refer to [Common Request Parameters](https://cloud.tencent.com/document/api/213/6976).

| Parameter Name | Required | Type | Description |
|---------|---------|---------|---------|
| queueName | Yes | String | Queue name. This is unique under the same account in one region. The name of queue is a string with no more than 64 characters. It must start with letter, and the rest may contain letters, numbers and dashes (-). |
| receiptHandle.n | Yes | String | Receipt handle returned when consuming message the last time. To make it more convenient for users, "n" may start from either 0 or 1, but is must be continuous. For example, if you delete two messages, they can be (receiptHandle.0,receiptHandle.1) or (receiptHandle.1, receiptHandle.2). |


## 3. Output Parameters

| Parameter Name | Type | Description |
|---------|---------|---------|
| code | Int | 0: Succeed, 4420: Maximum qps limit has been reached, 4440: Queue does not exist, 6010: Failed to delete some of the messages, 6020: Failed to delete any of the messages. For the meanings of other returned values, please refer to [Error Codes](/doc/api/431/5903). |
| message | String | Error message. |
| requestId | String | ID of the request generated by server. When there is an internal error on the server, users can submit this ID to backend to locate the problem. |
| errorList | Array | List of errors regarding failed deletion operations. Each element contains the error and reason for why a message could not be deleted. |

errorList is defined as follows:

| Parameter Name | Type | Description |
|---------|---------|---------|
| code | Int | 0: Succeeded, others: Error. For detailed errors, please refer to the table below. |
| message | String | Error message. |
| receiptHandle | String | Receipt handle of the message that was not successfully deleted. |

<table class="t">
<tbody><tr>
<th> <b>Error Code</b>
</th><th> <b>Module Error Code</b>
</th><th> <b>Error Message</b>
</th><th> <b>Description</b>
</th></tr>
<tr>
<td> 6010
</td><td> 10150
</td><td> delete message partially failed
</td><td> Failed to delete some of the messages in the batch. An error message will be provided for each of the messages that was not successfully deleted. Failed deletions are likely caused by invalid receipt handles.
</td></tr>
<tr>
<td> 4430
</td><td> 10260
</td><td> receipt handle is invalid
</td><td> Receipt handle is invalid. Refer to <a href="https://cloud.tencent.com/doc/api/431/5840">Delete Messages</a> for the reasons for why handles become invalid.
</td></tr>
<tr>
<td> 6020
</td><td> 10290
</td><td> batch delete message failed
</td><td> Failed to batch delete messages.
</td></tr>
<tr>
<td> 4000
</td><td> 10470
</td><td> receiptHandle error
</td><td> receiptHandle error. receiptHandle is a string
</td></tr>

</tbody></table>

Note: The error codes listed in the above table are specific to the API. If the error code you are looking for is not here, you may find it in the [Common Error Codes](https://cloud.tencent.com/document/product/406/5903).

## 4. Example

Input:

<pre>
 https://domain/v2/index.php?Action=BatchDeleteMessage
 &queueName=test-queue-123
 &receiptHandle.1=3423452345
 &receiptHandle.1=4364564575
 &<<a href="https://cloud.tencent.com/doc/api/229/6976">Common request parameters</a>>
</pre>

Output:

When all are successfully deleted

```
{
"code" : 0,
"message" : "",
"requestId":"14534664555"
}
```


When part of the messages are not successfully deleted

```
{
"code" : 6010,
"message" : "delete message partially failed",
"requestId":"14534664555",
"errorList":
[
{
"code" : 4430,
"message" : "invalid receiptHandle",
"receiptHandle":"4364564575"
}
]
}
```






