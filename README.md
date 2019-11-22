# SMSTimer
基于SpringBoot短信验证码定时任务相关:

```
定时器类:
@Component
@Configuration
@EnableScheduling
public class SMSTimer
{
     /*  存储用户的验证码 */
    public static Map<String, SmsDescribe> verificationCode = new HashMap<String, SmsDescribe>();


    //每60秒扫描一次 有没有过期的短信验证码
    @Scheduled(cron = "0 */1 * * * ?")
    private void SMSTask()
    {
        try {        
            Iterator<Map.Entry<String, SmsDescribe>> it = SMSTimer.verificationCode.entrySet().iterator();
            while (it.hasNext()) {
                Map.Entry<String, SmsDescribe> itEntry = it.next();
                SmsDescribe sd = itEntry.getValue();
                if(null != sd && null !=sd.getTime())
                {
                    long time = System.currentTimeMillis()-sd.getTime().getTime();
                    //本条记录大于了3分钟 删除数据
                    if(time>1000*180)
                    {
                        System.out.println("--正在清理短信验证码--");
                        it.remove();
                    }
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
}
```



```
/**
 * 短信验证码工具类
 * @Author:  no one
 * @Date:   2019-11-12 11:14:30
 * */
public class SmsDescribe {

	/**手机号*/
	private String phone;
	/**验证码 */
	private String smsCode;
	/**发送时间*/
	private Timestamp time;
	/** 图形验证码 */
	private String imgVerify;
	
	public String getPhone() {
		return phone;
	}
	public void setPhone(String phone) {
		this.phone = phone;
	}
	public String getSmsCode() {
		return smsCode;
	}
	public void setSmsCode(String smsCode) {
		this.smsCode = smsCode;
	}
	public Timestamp getTime() {
		return time;
	}
	public void setTime(Timestamp time) {
		this.time = time;
	}
	public String getImgVerify() {
		return imgVerify;
	}
	public void setImgVerify(String imgVerify) {
		this.imgVerify = imgVerify;
	}
	
	

}
```




# 具体使用: 
            
            //使用第三方短信服务,发送成功时,需要把验证码存储到内存中
            SmsDescribe sd = new SmsDescribe();
            sd.setPhone(phone);
            sd.setSmsCode(String.valueOf(smsCode));
            sd.setTime(new Timestamp(System.currentTimeMillis()));
            SMSTimer.verificationCode.put(phone, sd);


        其它地方调用的时候:
        
             // 首先获取当前手机号的验证码是否存在(定时任务每分钟遍历一次所有内存中的验证码,超过3分钟验证码被销毁)
             SmsDescribe sms_entity = SMSTimer.verificationCode.get(phone);
           
             //此手机号没有缓存的验证码 或者 验证码匹配不一致
             if (null == sms_entity || !sms_entity.getSmsCode().equals(mCode))
             {
                 //这里编写具体的业务逻辑
             }
            
           
            
#附录:定时任务Scheduled表达式 及 顺序输出sql示例

        /**结果按userId为 6,3,1,4,2顺序输出, 不在是默认的1,2,3,4,6输出, 满足部分业务场景需要 */ 
        select * from t_user where   t_id in (6,3,1,4,2)   ORDER BY field (6,3,1,4,2) 


	每隔5秒执行一次："*/5 * * * * ?"

	每隔1分钟执行一次："0 */1 * * * ?"

	每天23点执行一次："0 0 23 * * ?"

	每天凌晨1点执行一次："0 0 1 * * ?"

	每月1号凌晨1点执行一次："0 0 1 1 * ?"

	每月最后一天23点执行一次："0 0 23 L * ?"

	每周星期天凌晨1点实行一次："0 0 1 ? * L"

	在26分、29分、33分执行一次："0 26,29,33 * * * ?"

	每天的0点、13点、18点、21点都执行一次："0 0 0,13,18,21 * * ?"

	表示在每月的1日的凌晨2点调度任务："0 0 2 1 * ? *"

	表示周一到周五每天上午10：15执行作业："0 15 10 ? * MON-FRI" 

	表示2002-2006年的每个月的最后一个星期五上午10:15执行："0 15 10 ? 6L 2002-2006"
