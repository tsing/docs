---
order: 2
---

# 常见问题

1. [如何绑定云供应商API Key?](#id1)
2. [可以直接在青云或AWS的控制台删除NiceScale创建的资源吗？](#id2)
3. [NiceScale创建的虚拟硬件资源，在IaaS面板中如何识别？](#id3)
4. [项目内部的机器安全设置是怎样的？](#id4)


<a name="id1"></a>
### 如何绑定云供应商API Key? 

您可以在对应的IaaS服务商面板上查看到对应的API Key，并填写到NiceScale的“云供应商”里。


  AWS API Key
  
  ![AWS API Key](/assets/aws-apikey.png "AWS API Key")
  
  
  QingCloud API Key
  
  ![青云API Key](/assets/qing-apikey.png "QingCloud API Key")


<a name="id2"></a>
### 可以直接在青云或AWS的控制台删除NiceScale创建的资源吗？

可以但不推荐，您应当从NiceScale的控制台删除由NiceScale创建的资源，这样可以避免不一致。

<a name="id3"></a>
### NiceScale创建的虚拟硬件资源，在IaaS面板中如何识别？

IaaS面板中的资源名称如果以"ns-"开头，那么就是NiceScale代创建的。

<a name="id4"></a>
### 项目内部的机器安全设置是怎样的？

NiceScale的项目，是部署在你的某个VPC下的一个子网(subnet)中的。同一个项目的所有机器总是在一个子网内部，通过私网IP地址可以直接访问，端口也没有任何限制。但是对外部，则是通过安全组SecurityGroup或防火墙进行防御，默认开启的端口一般是登录用途，内部服务的端口外部无法访问到，如需要对外暴露，请自行到IaaS面板里进行设置。

