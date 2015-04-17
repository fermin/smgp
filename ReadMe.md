#本项目说明

# SMGP API 说明 #

> SMGP协议是中国电信为短信增值业务指定的接口协议，协议在在SMPP协议基础上进行扩展，扩展了相关计费和鉴权信息。目前SMGP协议有1.0，2.0，3.0三个版本的协议。1.0协议目前基本上停用，而2.0目前在部分省小灵通接入仍然在使用。3.0协议时目前最新的协议，相比2.0协议主要增加了tlv可选字段。 相关类说明


# API使用简介 #

## 1. 类说明 ##
### 1.1 Client类 ###
类命名空间：cn.com.zjtelecom.smgp<br />
public Client(String Host, int port, int LoginMode, String clientid,<br />
> String ClientPasswd, String SPID)
<br />
说明：初始化smgp类<br />
Host:短信网关ip地址<br />
Port:短信网关端口<br />
LoginMode：登录模式，0：单发 1：单收 2：收发，如果有一个类已经启动为单收或者收发，另外一个类使用单收或者收发，只有第一个类才能收到Deliver消息<br />
ClientId：网关帐号<br />
ClientPasswd：网关密码<br />
SPID：ISMP上的企业代码，长度8位<br />

public Client(String Host, int port, int LoginMode, String clientid,<br />
> String ClientPasswd, String SPID, int DisplayMode)<br />
说明：初始化smgp类<br />
Host:短信网关ip地址<br />
Port:短信网关端口<br />
LoginMode：登录模式，0：单发 1：单收 2：收发，如果有一个类已经启动为单收或者收发，另外一个类使用单收或者收发，只有第一个类才能收到Deliver消息<br />
ClientId：网关帐号<br />
ClientPasswd：网关密码<br />
SPID：ISMP上的企业代码，长度8位<br />
DisplayMode：0不显示信息，1显示必要信息，2显示除ActiveTest以外的封包信息，3显示所有的封包信息<br />

public void setDisplayMode(int DisplayMode)<br />
说明：设置显示模式<br />
DisplayMode：0不显示信息，1显示必要信息，2显示除ActiveTest以外的封包信息，3显示所有的封包信息<br />

public Result Login() <br />
说明：查询登录结果<br />
如返回 Result.ErrorCode非0表示登录失败 <br />

public Deliver OnDeliver()<br />
说明：查询上行消息<br />
该方法为阻塞方式调用，直到有上行消息才会返回。<br />

public Deliver getDeliver()<br />
说明：查询上行消息<br />
该方法为非阻塞方式调用，如当时无上行消息会返回空值。<br />

public Result Send(Submit submit) <br />
说明：发送消息<br />

### 1.2 Submit类 ###
public Submit()<br />
说明：初始化一个类<br />
类会使用初始化值创建submit类<br />
缺省值如下<br />
```
MsgType = 6;
NeedReport = 0;
Priority = 0;
ServiceID = "";
Feetype = "00";
FeeCode = "000";
FixedFee = "000";
MsgFormat = 15;
ValidTime = "";
AtTime = "";
SrcTermid = "";
DestTermid = "";
ChargeTermid = "";
MsgLength = 0;
MsgContent = null;
Reserve = null;
ProductID = "";
LinkID = "";
OtherTlvArray = null;
```
Submit本身为一个java bean,可以通过set变量名，或者get变量名的方法，设置和获取<br />

public Submit(int msgtype, int needreport, int priority, String serviceid, String feetype, String feecode, String fixedfee, int msgformat, String validtime, String attime, String srctermid, String desttermid, String chargetermid, int msglength, byte[.md](.md) msgcontent, byte[.md](.md) reserve, String productid, String Linkid)<br />
说明：初始化一个类，并设置值<br />
Msgtype：消息类型，MT消息统一设为6<br />
Needreport:是否需要状态报告<br />
Priority：消息优先级<br />
Serviceid：服务代码，在浙江的电信的ismp平台，该代码无效<br />
Feetype：计费类型，免费设00,按次设01，包月是02，封顶03<br />
Feecode：按次费用<br />
Fixedfee：包月费用，如果Feetype为03<br />
Msgformat：消息编码，0为ASCII,15为GBK编码，8位UCS-2编码<br />
Validtime：消息有效时间<br />
Attime：定时发送时间<br />
Srctermid：源地址号码<br />
Desttermid：目的号码<br />
Chargetermid：计费号码<br />
Msglength：消息长度<br />
Msgcontent：消息内容<br />
Reserve：保留字段<br />
Productid：产品编号<br />
Linkid：点播下发时需要将deliver消息里面的linkid作为消息的Linkid下发<br />

public Submit(String srctermid, String desttermid, byte[.md](.md) msgconent)<br />
说明：初始化一个类，并设置值，未设置的值为初始值<br />
Srctermid：源地址号码<br />
Desttermid：目的号码，计费号码会使用Desttermid替换<br />
Msgconent：消息体，格式为byte数组<br />


public Submit(String srctermid, String desttermid, String msgconent)<br />
说明：初始化一个类，并设置值，未设置的值为初始值<br />
Srctermid：源地址号码<br />
Desttermid：目的号码，计费号码会使用Desttermid替换<br />
Msgconent：消息体，格式为字符串<br />

public Submit(String srctermid, String desttermid, String msgconent,<br />
> String productid)<br />
说明：初始化一个类，并设置值，未设置的值为初始值<br />
Srctermid：源地址号码<br />
Desttermid：目的号码，计费号码会使用Desttermid替换<br />
Msgconent：消息体，格式为字符串<br />
Productid：产品编号<br />


public Submit(String srctermid, String desttermid, String msgconent,
> String productid)<br />
说明：初始化一个类，并设置值，未设置的值为初始值<br />
Srctermid：源地址号码<br />
Desttermid：目的号码，计费号码会使用Desttermid替换<br />
Msgconent：消息体，格式为字符串<br />
Productid：产品编号<br />

public Submit(String srctermid, String desttermid, byte[.md](.md) msgconent,
> String productid)<br />
说明：初始化一个类，并设置值，未设置的值为初始值<br />
Srctermid：源地址号码<br />
Desttermid：目的号码，计费号码会使用Desttermid替换<br />
Msgconent：消息体，格式为byte数组<br />
Productid：产品编号<br />

public void AddTlv(int tag, String value)<br />
说明:tlv字段，srcMsg是企业代码，在初始化client时候统一设置无需这里单独设置，另外Productid和linkid可以直接设置，也无需使用该方法设置<br />
Tag:tlv tag标记<br />
Value：值<br />

## 2. api使用 ##
### 2.1 发送消息 ###
```
Client client = new Client("189.189.189.189", 8890, 0, "与网关连接帐号",
				"与网关连接密码", "ISMP上的企业代码");
Submit submit = new Submit("10620068", "18967441118", "Test中文",
				"112000000000061090527");
Result result=client.Send(submit);
```

### 2.2 接收消息 ###
#### 2.2.1 非等待方式 ####
```
Client client = new Client("189.189.189.189", 8890, 1, "与网关连接帐号",
				"与网关连接密码", "ISMP上的企业代码");
Deliver dl=client.getDeliver();
if (dl!=null){		 System.out.println("---------------Deliver---------------");
			System.out.println("SrcTermID:" + dl.SrcTermID);
			System.out.println("DestTermID:" + dl.DestTermID);
			System.out.println("MsgContent:"
					+ new String(dl.MsgContent));
			System.out.println("LinkID:" + dl.LinkID);
			System.out.println("-------------------------------------");
		}else {
			System.out.println("no deliver");
		}
```

#### 2.2.2 等待方式 ####
```
Client client = new Client("189.189.189.189", 8890, 1, "与网关连接帐号",
				"与网关连接密码", "ISMP上的企业代码");	
Deliver dl = client.OnDeliver();
System.out.println("---------------Deliver---------------");
					System.out.println("SrcTermID:" + dl.SrcTermID);
					System.out.println("DestTermID:" + dl.DestTermID);
					System.out.println("MsgContent:"
							+ new String(dl.MsgContent));
					System.out.println("LinkID:" + dl.LinkID);
		
System.out.println("-------------------------------------");
```