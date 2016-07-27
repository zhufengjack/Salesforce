# EFEC Salesforce Logic
## Overview Flow of Salesforce
如下图所示，我们各个系统之间的集成图。简单来说，Marketing team通过在不同网站上的landing
page收集潜在客户信息，然后存入SQL Server中ET_Payment中的AgentRequest and AgentRequestExtension
这两张表中。然后由部署在我们自己Server上的Windows Service（SPOP）LAS 抓取AgentRequest
和AgentRequestExtension
中的信息，push到Salesforce里边（最大批次为100）

```
graph TD
  subgraph Marketing
  A[Landing Page]--Leads Handler-->B((Agent Request))
  end

  subgraph LAS
  C[Leads Automation System]--Pull Leads information-->B
  end

  subgraph Salesforce
  C -->D[Leads in Salesforce]
  end
```


### LAS 抓取Lead规则如下：
#### EFEC CN
#### EFEC INDO
#### EFEC RU
#### EFEC SP
#### EFTS CN
#### EFEC HK
## Salesforce的数据初始化
在leads信息被push到Salesforce之后，Salesforce会进行数据初始化，按照Lead的Country
和Business Unit给Leads的Record Type和Currency进行赋值，并验证数据的有效性，例如电话号码
是否合法等。
#### Leads Record Type 和 Currency 初始化
[Lead Record Tpye 分配规则](https://ap1.salesforce.com/setup/ui/listCustomSettingsData.apexp?id=a08)
**Name=Country Code + Business Unit**
Name | Record Type | Currency
---|---|---
cn_mini   | EFEC_B2C_CN_Leads | CNY
cn_smart  | EFEC_B2C_CN_Leads| CNY
cn_minits | EFTS_B2C_CN_Leads| CNY
cn_ts     | EFTS_B2C_CN_Leads| CNY
es_ec | EFEC_B2C_SP_Leads| EUR
id_e1 | EFEC_B2C_ID_Leads| IDR
ru_e1 | EFEC_B2C_RU_Leads| RUB

#### 数据有效性检验
##### EFEC CN AND EFTS CN
对中国的Leads，不管是TS还是EC，首先进行MobilePhone的验证，如果MobilePhone不合法，那么
验证Phone字段。具体逻辑如下图所示
```
graph TD
A[去除电话号码中的非数字字符比如括号横线] -->B[将去除特殊字符的电话号码赋值给MobilePhone 和Phone]
B-->C[检查电话号码的长度如果长度小于5那么Phone Valid 为False]
C-->D[检查该电话号码是否在黑名单里边 如果是 那么Phone Valid为False]
D-->E[检查电话号码的前3位是否在预设好的list里边]

```

##### EFEC INDO AND RU
对于俄罗斯和印度尼西亚的Lead，Phone和mobile的验证，只要长度大于6，我们就认为号码是有效的。并且，如果Mobile为空，则把Phone赋值给Mobile。如果Phone为空，则把Mobile赋值给Phone。

#### EFEC SP
对于西班牙的Leads，系统会检验电话号码是否在黑名单里边，并且验证开始数字是否已6，7,8,9开头。如果电话号码以0034开头，那么系统自动去除0034. 最后
如果Mobile为空，则把Phone赋值给Mobile。如果Phone为空，则把Mobile赋值给Phone。





## Leads De-duplication Logic

各个国家和BU的查重规则各不相同，具体如下：

### EFEC CN De-duplication Logic


```
graph TD
A[De-duplicate in same batch]-->B[De-duplicate with existed Leads]
B-->C[De-duplicate with existed Opportunities]

```
