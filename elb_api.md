# ELB api文档

## 目录

* 说明
* 用户API -- 操作类
	* CreateLoadBalancer
	* CheckLBStatus
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

返回 CreateLoadBalancerStatus

* Status:  
	* 创建状态, 一般返回开始创建状态
	* Type: Integer

#### 错误

##### DuplicateAccessPointName
	
* 名称重复
* HTTP Status Code: 400

##### InvalidateParamRequest

* 请求参数错误
* HTTP Status Code: 409


####  示例

* 示例请求

```
https://elb.cloud.netease.com/api?LoadBalancerName=blog
&Scheme=http&InstanceNum=5&InstancePort=8083
```

* 示例响应

```
{
	"Status" : 1
}
```


### 2.1 CheckLBStatus

	



	 	 