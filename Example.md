#API使用例子.

# 1.使用cn.com.zjtelecom.smgp.Client #

## 1.1登录 ##
```
//Client("服务器ip地址","服务器端口号",LoginMode,"账号","密码","企业代码",显示模式)
Client client = new Client("189.189.189.189", 8890, 2, "account","password", "12100001", 0);
Result result = client.Login();
System.out.println("ErrorCode:"+result.ErrorCode);
System.out.println("ErrorDescription:"+result.ErrorDescription);
```

调用client.Login()非必须方式，如果程序登录没有问题，无须先调用Login,api本身会自动调用

## 1.2发送 ##
```
//Client("服务器ip地址","服务器端口号",LoginMode,"账号","密码","企业代码",显示模式)
Client client = new Client("189.189.189.189", 8890, 2, "account","password", "12100001", 0);
//submit("主叫号码","被叫号码","内容","产品编号")
Submit submit = new Submit("10620068", "18967441118", "你好！","112000000000061090527"); Result result = client.Send(submit);
System.out.println("ErrorCode:"+result.ErrorCode);System.out.println("MsgID:"+result.ErrorDescription);
```
## 1.3发送长短信 ##
使用Client.SendLong(Submit)方法可以直接发送长短信

```
                Client client = new Client("网关ip地址", 8890, 2, "网关账号",
                                "网关密码", "企业代码", 0);
                Submit submit = new Submit("10620068", "15305711873", "浙江在线07月12日讯 7月11日，正在葡萄牙参加一个重要经贸洽谈会的温州人叶茂西证实，英国本土一家卫星电视台——PROPELLER(译为螺旋桨)电视台已被他旗下的西京集团有限公司全资收购。\nPROPELLER是欧洲第一个100%放映全新的原创节目的数字卫星电视频道，通过英国天空广播公司的卫星平台传送，由英国格林斯比研究所附属公司图像频道有限公司于2006年2月创办，它最早在英国约克郡播放，后来在欧洲数十个国家落地。2008年11月，该电视台在意大利威尼斯获得“全欧洲最佳卫视电影频道”奖项。\n叶茂西是平阳人，现任北京温州商会会长、中国丝网印刷协会理事长、西京集团有限公司董事长等职。今年初，在温家宝总理访英期间，叶茂西随中国经贸代表团出访英国。其间，叶茂西了解到，金融危机使英国的这家电视台不能再获得政府资助，急需引进战略投资人。当时，叶茂西以西京集团的名义与该电视台达成初步收购意向。经过近半年努力，上月，西京集团斥巨资完成了对该电视台的收购工作。".getBytes("iso-10646-ucs-2"),
                "112000000000061090527");
                submit.setMsgFormat(8);
                Result [] result= client.SendLong(submit);
                
                for(int i=0;i<result.length;i++){
                        System.out.println("result:"+result.ErrorCode);
                        System.out.println("msgid:"+result.ErrorDescription);
                        
                }
```

## 1.4接收 ##
```
Client client = new Client("短信网关ip地址", 8890, 2, "短信网关账号", "短信网关密码",
        "企业代码（8位长）", 0); // new Client("IP地址",端口,登录模式,"帐号","密码")

Result result = client.Login();
System.out.println("---------------Login---------------");
System.out.println("Code:" + result.ErrorCode);
System.out.println("DEsc:" + result.ErrorDescription);
System.out.println("-----------------------------------");


if (result.ErrorCode == 0) {   //登录成功
int i = 1;
do {
        Deliver dl = client.OnDeliver();

        if (dl.IsReport != 1) {
                System.out.println("---------------Deliver---------------");
                System.out.println("SrcTermID:" + dl.SrcTermID);
                System.out.println("DestTermID:" + dl.DestTermID);
                System.out.println("MsgContent:"
                                + new String(dl.MsgContent));
                System.out.println("LinkID:" + dl.LinkID);
                System.out.println("-------------------------------------");

                //上行处理程序扔这里
                              }
     }while(true);
  }

```

# 2.事件触发方式 #

由于cn.com.zjtelecom.smgp.Client在提交Submit消息时，等待Submit\_Resp，导致发送速度较慢。ClientEvent类修改为发送Submit消息时，只返回Submit包的SeqID,而Submit\_Resp和Deliver通过调用程序实现接口ClientEventInterface的方法，ClientEvent通过onDeliver和OnSubmitResp来回调该方法，通知程序收到Submit\_resp和上行消息


# 2.1例子 #


```
import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.UnsupportedEncodingException;
import java.util.HashMap;

import cn.com.zjtelecom.smgp.ClientEvent;
import cn.com.zjtelecom.smgp.Exception.SubmitException;
import cn.com.zjtelecom.smgp.bean.Deliver;
import cn.com.zjtelecom.smgp.bean.Result;
import cn.com.zjtelecom.smgp.bean.Submit;
import cn.com.zjtelecom.smgp.bean.SubmitResp;
import cn.com.zjtelecom.smgp.inf.ClientEventInterface;
import cn.com.zjtelecom.stock.Stock;

public class StockTest {
	/*
	 * Config类为获取配置文件类，系统缺省读取当前目录下的smgpc.ini
	 * 配置文件应包含以下参数
	 * smgwip=短信网关ip地址
	 * smgwport=短信网关端口
	 * smgwaccount=账号
	 * smgwpasswd=密码
	 * smgwspid=企业代码
	 * 
	 * */
	private static class Config {
		 private static String configFile="smgpc.ini";
		 private  HashMap <String,String> configproperty=new HashMap<String,String>();
		 
	   public Config() throws IOException {
	  	 this.ReadProperty(configFile);
	   }
		 public  Config(String configFile2) throws IOException {
			// TODO Auto-generated method stub
			 this.ReadProperty(configFile2);
		}
		private  void ReadProperty(String cfile) throws IOException{
				String line = "";
				FileInputStream fileinputstream = new FileInputStream(cfile);
				BufferedReader bufferedreader = new BufferedReader(
						new InputStreamReader(fileinputstream));
				do {
					if ((line = bufferedreader.readLine()) == null)
						break;
					if (line.indexOf("#") == 0 || line.indexOf("=") < 0)
						continue;
					String key=line.substring(0,line.indexOf("="));
					String value=line.substring(line.indexOf("=")+1);
					
					//System.out.println("key:"+key);
					//System.out.println("value:"+value);
					configproperty.put(key,value);
					
				} while(true);
		 }
		public  String get(String key){
			return configproperty.get(key);
		}
	}
	
	/*
	 * SmgpClient实现接口ClientEventInterface的方法,ClentEvnet会将
	 * OnSubmitResp，onDeliver回调给该类
	 * 
	 * */
	private static class SmgpClient implements ClientEventInterface {
		public ClientEvent client = null;

		public SmgpClient(String host, int port, int loginmode,
				String clientid, String clientpasswd, String spid,
				int displaymode) {
			this.client = new ClientEvent(this, host, port, loginmode,
					clientid, clientpasswd, spid, displaymode);
		}

		public void OnSubmitResp(SubmitResp submitResp) {
			System.out.println("-----------------GetSubmitResp-----------------");
			System.out.println("ResultCode:" + submitResp.getResultCode());
			System.out.println("MsgID:" + submitResp.getMsgID());
			System.out.println("SequenceID:" + submitResp.getSequenceID());
			System.out.println("DestNum:"
					+ submitResp.getSubmit().getDestTermid());

		}

		public void onDeliver(Deliver deliver) {
			System.out.println("-----------------GetDeliver-----------------");
			System.out.println("IsReport:" + deliver.IsReport);
			System.out.println("DestTermID:" + deliver.DestTermID);
			System.out.println("SrcTermID:" + deliver.SrcTermID);
			System.out.println("LinkID:" + deliver.LinkID);
			System.out.println("ReportMsgID:" + deliver.ReportMsgID);
			System.out.println("MsgContent:" + new String(deliver.MsgContent));

			if (deliver.IsReport == 0) {
				try {
					if (new String(deliver.MsgContent, "ISO8859-1").substring(
							0, 1).equalsIgnoreCase("s")) { // 如果是s开头的指令，表示用户是点播的股票业务
						String stockstr = Stock.getStock(new String(
								deliver.MsgContent, "ISO8859-1"));//该方法为获取股票数据，需要自行实现
						Submit submit = new Submit("10620068",
								deliver.SrcTermID, stockstr.getBytes("iso-10646-ucs-2"),
								"112000000000000001032");
						submit.setFeetype("01");
						submit.setFeeCode("0");
						submit.setMsgFormat(8);
						submit.setLinkID(deliver.LinkID);
						submit.setNeedReport(1);
						submit.setChargeTermid(deliver.SrcTermID);
						Integer seq[] = client.SendLong(submit);
						for (int i=0;i<seq.length;i++){
						System.out.println("SubmitSeq:"+seq[i]);
						}
					}

				} catch (UnsupportedEncodingException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				} 
				catch (SubmitException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}

		}

	}

	public static void main(String[] args) throws IOException {
		Config configproperty=new Config();
		SmgpClient smgpclient = new SmgpClient(configproperty.get("smgwip"),
				Integer.parseInt(configproperty.get("smgwport")), 2,
				configproperty.get("smgwaccount"), configproperty
						.get("smgwpasswd"), configproperty
						.get("smgwspid"), 0);
		smgpclient.client.setSpeed(10); // 设置发送速度，单位为 条/秒
		Result loginresult= smgpclient.client.Login();
		System.out.println("LoginStatus:"+loginresult.ErrorCode);
		System.out.println("LoginDescription:"+loginresult.ErrorDescription);
		
		if (loginresult.ErrorCode!=0) {
			System.out.println("登录失败");
			smgpclient.client.Close(); //登录失败关闭client的链接，以便程序正常退出。
		} else {
			System.out.println("登录成功");
		}

	}

}

```