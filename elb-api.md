# ELB api文档

## 目录
* 说明
* 用户API -- 操作类
	* CreateLoadBalancer
	* CheckLoadBalancerStatus
	* DeleteLoadBalancer
	* ConfigureHealthCheck
	* CreateLBCookieStickinessPolicy
	* CreateLoadBlanacerPolicy
	* DeleteLoadBalancerPolicy
	* RegisterInstancesWithLoadBalancer
	* DeregisterInstancesFromLoadBalancer
	* ModifyLoadBalancerAttributes
	* SetLoadBlanacerListnerSSLCertificate
	* SetLoadBlanacerPoliciesForBackendServer
	* SetLoadBalancerPoliciesOfListener	
	
* 用户API -- 描述类
	* DescribeLoadBlanacerAttributes
	* DescribeInstanceHealth
	* DescribeLoadBalancerPolicies
	* DescribeLoadBalancerPolicyTypes
	* DescribeLoadBalancers
	 
* 管理员API -- 操作类
	* RegisterTengineWithLoadBalancer
	* DeregisterTengineFromLoadBalancer
	* UpdateAgentForLoadBalancer
	* UpdateScriptForLoadBalancer
		
* 管理员API -- 描述类	
	* DescribeLoadBalancerServers
	* DescribeLoadBalancerLVSStatus
	* DescribeLoadBalancerTengineStatus
	* DescribeLoadBalancerLVSConf
	* DescribeLoadBalancerTengineConf
	
## 一、说明

* 目前所有API不做auth校验，只在api server上设白名单
* 所有请求走https， 默认域名 https://elb.cloud.netease.com/api
* 所有返回结果都是json
* 所有响应结果里都带了requestId，参考了aws，主要为了日志跟踪，所有请求都有唯一的id

## 二、用户API -- 操作类

### 2.1 CreateLoadBalancer

#### 描述

创建一个负载均衡器。

该操作周期较长，因此请求提交后，服务端校验请求是否合法，然后直接返回。 

后续通过CheckLbCreateStatus检查创建的状态与结果。

#### 请求参数

* LoadBalancerName： 名称
* Scheme: http 或 tcp
* InstanceNum: 后端instance数量
* InstancePort: 后端instance端口

#### 响应结果

* Status:  
	* 创建状态, StartCreate, ProcessCreate, CreateSuccess, CreateFail
	* Type: String

#### 错误

##### DuplicateAccessPointName
	
* 名称重复
* HTTP Status Code: 400

##### InvalidateParamRequest

* 请求参数错误
* HTTP Status Code: 409

##### InvalidateScheme

* 传入Scheme错误
* HTTP Status Code: 409

####  示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=CreateLoadBalancer
&LoadBalancerName=blog&Scheme=http
&InstanceNum=5&InstancePort=8083
```

* 示例响应

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",
	"Status" : "StartCreate"	
}
```

* 示例错误响应

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",
	"error" : "DuplicateAccessPointName"	
}

```

### 2.1 CheckLoadBalancerStatus

#### 描述

检查ELB的创建状态

#### 请求参数

* LoadBalancerName: 负载均衡器名称

#### 响应结果

* Status:  
	* 创建状态, StartCreate, ProcessCreate, CreateSuccess, CreateFail	* Type: String
* Step：
	* 创建步数：到工作流的第几步
	* Type: Integer
* Url
	* 创建成功后的url， 不成功没有这个参数
	* Type: String
	
#### 错误
	
##### InvalidateParamRequest

* 请求参数错误
* HTTP Status Code: 409

##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=CheckLoadBalancerStatus
&LoadBalancerName=blog

```
* 示例响应

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",
	"status": "CreateSuccess",
	"step"": 8,
	"url" : "http://10.123.123.123:7070"
}
```

### DeleteLoadBalancer

#### 描述

删除负载均衡器。

这个过程也可能比较久， 所以校验完参数即返回， 通过CheckLoadBalancerStatus确定是否成功。

#### 请求参数

* LoadBalancerName: 负载均衡器名称

#### 响应结果

* Status
	* 开始进行：StartDelete
	* Type: String

#### 错误
	
##### InvalidateParamRequest

* 请求参数错误
* HTTP Status Code: 409

##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=DeleteLoadBalancer
&LoadBalancerName=blog

```
* 示例响应

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",
	"status": "DeleteSuccess"
}
```


### ConfigureHealthCheck

#### 描述

配置健康检查的参数

#### 请求参数

* LoadBalancerName: 负载均衡名
* HealthCheck: 健康检查的参数信息，包括以下字段：


#### 响应结果

* HealthCheck： 更新新后的健康检查结果

#### 错误
	
##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=ConfigureHealthCheck
&LoadBalancerName=blog
&HealthCheck.HealthyThreshold=2
&HealthCheck.HealthyThreshold=2
&HealthCheck.Target=HTTP:80/ping
&HealthCheck.Interval=30&HealthCheck.Timeout=3

```
* 示例响应

```

{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",
	"HealCheck": {
		"Integerval": 30,
		"Target": HTTP:80/ping,
		"HealthyThreshold":2,
		"Timeout":3,
		"UnhealthyThreshold":2
	}
}

```



### CreateLBCookieStickinessPolicy

#### 描述

创建session sticky的policy。

#### 请求参数

* LoadBalancerName: 负载均衡名
* CookieExpirationPeriod: cookie超时时间，默认会持续到浏览器关闭
* CookieName：负载均衡基于cookie名， 默认为jsessionid
* PolicName：策略名


#### 响应结果

只需要requestId

#### 错误
	
##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

##### DuplicatePolicyName

* 策略名重复
* HTTP Status Code: 400

##### InvalidConfigurationRequest

* 请求的配置参数不正确
* HTTP Status Code: 409


#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=CreateLBCookieStickinessPolicy
&LoadBalancerName=blog
&CookieName=xsessionid
&PolicyName=blogcookiePolicy

```
* 示例响应

```

{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE"
}

```

### CreateLoadBalancerPolicy

#### 描述

创建策略。 策略可认为是某类选项，如请求超时的选择、tengine选项， LVS选项等。如：

* SSLNegotiationPolicyType
* BackendServerAuthenticationPolicyType
* PublicKeyPolicyType
* LBCookieStickinessPolicyType
* ProxyProtocolPolicyType

#### 请求参数

* LoadBalancerName: 负载均衡名
* PolicName：策略名
* PolicyTypeName： 对应的策略类型
* PolicyAttributes.member.N: 一系列的策略参数


#### 响应结果

只需要requestId

#### 错误
	
##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

##### DuplicatePolicyName

* 策略名重复
* HTTP Status Code: 400

##### InvalidConfigurationRequest

* 请求的配置参数不正确
* HTTP Status Code: 409

##### PolicyTypeNotFound* PolicyType不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=CreateLoadBalancerPolicy
&Name=ProxyProtocol&PolicyAttributes.member.1.AttributeValue=true &PolicyTypeName=ProxyProtocolPolicyType 
&LoadBalancerName=my-test-loadbalancer&PolicyName=EnableProxyProtocol
```
* 示例响应

```

{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE"
}

```

### DeleteLoadBalancerPolicy

#### 描述

删除策略。 

#### 请求参数

* LoadBalancerName: 负载均衡名
* PolicyName: 策略名称

#### 响应结果

只需要requestId

#### 错误
	
##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

##### DuplicatePolicyName

* 策略名重复
* HTTP Status Code: 400

##### PolicyNameNotFound* PolicyName不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=DeleteLoadBalancerPolicy
&LoadBalancerName=my-test-loadbalancer&PolicyName=EnableProxyProtocol
```
* 示例响应

```

{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE"
}

```

### RegisterInstanceWithLoadBalancer


#### 描述

往负载均衡器添加后端服务实例。

#### 请求参数

* Instances.member.N： instanceID列表。在网易应该是云主机的id
* LoadBalancerName：负载均衡器名称

#### 响应结果

* Instances： 更新的instance列表


#### 错误
	
##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=RegisterInstanceWithLoadBalancer
&LoadBalancerName=my-test-loadbalancer&Instances.memeber.1.InstanceId=i-315b7e51
```

* 示例响应

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",
	"Instances": [
		{"InstanceId": "i-315b7e51"},
		{"InstanceId": "i-315b7e87"},
	]
}
```

### DeregisterInstanceFromLoadBalancer


#### 描述

从负载均衡器添移除后端服务实例。

#### 请求参数

* Instances.member.N： instanceID列表。在网易应该是云主机的id
* LoadBalancerName：负载均衡器名称

#### 响应结果

* Instances： 更新的instance列表


#### 错误
	
##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=DeregisterInstanceFromLoadBalancer
&LoadBalancerName=my-test-loadbalancer&Instances.memeber.1.InstanceId=i-315b7e51
```

* 示例响应

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",
	"Instances": [
		{"InstanceId": "i-315b7e87"}
	]
}
```

### ModifyLoadBalancerAttributes

#### 描述 

修改负载均衡器的相关属性

#### 请求参数

* LoadBalancerName：负载均衡器名称
* LoadBalancerAttributes： 负载均衡器的属性，包括以下几项：
	* AccessLog：打开请求日志
	* ConnectionDrain: 等请求都发完后才将请求从要移除的实例移除
	
#### 响应结果

 * LoadBalancerName: 负载均衡器名称
 * LoadBalancerAttributes：负载均衡器的属性
 
#### 错误
	
##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### InvalidConfigurationRequest

* 请求配置的更改有错
* * HTTP Status Code: 409

##### LoadBalancerAttributeNotFound

* 请求的属性不存在
* HTTP Status Code: 400

## 示例


* 示例请求

```
https://elb.cloud.netease.com/api?action=ModifyLoadBalancerAttributes
&LoadBalancerName=my-test-loadbalancer&LoadBalancerAttributes.AccessLog.Enabled=true&LoadBalancerAttributes.AccessLog.S3BucketName=my-loadbalancer-logs&LoadBalancerAttributes.AccessLog.S3BucketPrefix=my-bucket-prefix/prod&LoadBalancerAttributes.AccessLog.EmitInterval=60
```

* 示例响应

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",
	"AccessLog": {
		 "Enabled": true,        "S3BucketName": "my-loadbalancer-logs",        "S3BucketPrefix": "my-bucket-prefix/prod",        "EmitInterval": 60
	}
}


















	



	 	 