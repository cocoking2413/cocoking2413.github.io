##登录测试逻辑流程

####基础流程
1. 访问登录页面[login](http://localhost:51290/Login)
2. 输入登录名
3. 输入密码
4. 登录（目前可能会因为没有菜单权限服务器报错）
5. 成功

####成功标志
转工作台页面

####关注
1. 数据校验
2. 交互提示
3. 账户锁定
4. 密码可见
5. 键盘事件

####数据校验
1. 用户名（目前支持用户号（6~10位）和手机号登陆）  
    6位以上数字，（暂时）不超过11位  
2. 密码    
	8~20位字母数字混合
####账户锁定
1. 用户超过4次输入不正确密码会锁定账户
2. 第四次会提示


##找回密码测试逻辑流程

####基础流程
1. 经登录页面访问找回密码页面[ReBackPwd](http://localhost:51290/Login/Login/GetBackPassWord)
2. 输入手机号
3. 手机号存在获取验证码
4. 填写验证码
5. 设置新密码
6. 重新输入新密码
7. 点击完成提交
8. 成功

####成功重置标志
1. 登录无误
2. 账户解锁

####关注
1. 数据校验
2. 交互提示
3. 键盘事件
4. 用户手机校验
5. 短信发送
6. 重置密码+解除锁定

####数据校验
1. 手机号  
    11位  
2. 密码    
	8~20位字母数字混合  
3. 验证码  
	4-6位数字


##注册测试逻辑流程

####基础流程（注册新企业）
1. 经登录页面访问注册页面[Register](http://localhost:51290/Login/Login/Register)
2. 输入手机号
3. 手机号可用获取验证码
4. 填写验证码
5. 设置用户姓名
6. 输入密码
7. 设置公司名称
8. 点击注册提交
9. 成功

####基础流程（加入企业--手动输入企业号和邀请码）
1. 经登录页面访问注册页面[Register](http://localhost:51290/Login/Login/Register)
2. 点击加入已有企业
3. 输入手机号
4. 手机号可用获取验证码
5. 填写验证码
6. 设置用户姓名
7. 输入密码
8. 填写邀请码
9. 填写企业码
10. 点击注册提交
11. 成功

####基础流程（加入企业--通过链接）
1. 经登录页面访问注册页面[Register](http://localhost:51290/Login/Login/Register)
2. 修改url路径，添加参数？link=@GUID,@GUID通过数据库base_InviteCode表查询一个可用(state=0)GUID,测试注意区分已有用户和新用户
3. 手机号可用获取验证码
4. 填写验证码
5. 设置用户姓名
6. 输入密码
7. 点击注册提交
8. 成功



####关注
1. 数据校验
2. 交互提示
3. 键盘事件
4. 用户手机校验
5. 短信发送
6. link方式自动处理（固定手机号，邀请码，企业码，显示加入企业）
8. 姓名+企业名数据校验（sql注入）

####数据校验
1. 手机号  
     11位
2. 密码    
	8~20位字母数字混合
3. 验证码  
	4-6位数字
4. 邀请码  
	6~8位数字字母混合
5. 企业码  
	7位以上数字
6. 姓名
	数字、字母、特殊字符、汉字（范围待议），1-20字符
7. 公司名
	数字、字母、特殊字符、汉字（范围待议），1-100字符

####验证码获取

base_SendMessage

type:
1. 注册
2. 找回密码

取[Mobile]=注册手机号，[State]=0，[MessageType]=相应type
的[VerifyCode]

####企业码获取
[base_ZMSCompany]
取相应
[CompanyNO]

####邀请信息获取
1. 数据库造邀请数据   

		declare @guid varchar(50)  --?link=guid
		declare @phone varchar(50)  --受邀请人手机号
		declare @invitecode varchar(50)--邀请码（数字加字母6-8）
		declare @comCode varchar(50)--企业号
		declare @comID int--公司ID
		declare @comName varchar(50)--公司名
		
		select @guid='34234vsd234234sdv4554'
		select @phone='17865915869'
		select @invitecode='34dsdf34'
		select @comCode='123'
		select @comID=3
		select @comName=N'智博星'
		
		INSERT INTO [zms3.02].[dbo].[base_InviteCode]
		           ([InviteGUID]
		           ,[InviteType]
		           ,[Email]
		           ,[Mobile]
		           ,[InviteeName]
		           ,[InviteCode]
		           ,[CompanyNO]
		           ,[ZMSCompanyID]
		           ,[CompanyName]
		           ,[State]
		           ,[InviteTime]
		           ,[VerifyTime]
		           ,[CreateTime])
		     VALUES
		           (@guid
		           ,0
		           ,''
		           ,@phone
		           ,''
		           ,@invitecode
		           ,@comCode
		           ,@comID
		           ,@comName
		           ,0
		           ,GETDATE()
		           ,GETDATE()
		           ,GETDATE())
		GO



2. @GUID  
	[base_InviteCode]取相应[[InviteGUID]]
3. 邀请码+企业号  
	[base_InviteCode]取相应[[[InviteCode]
      ,[CompanyNO]]]