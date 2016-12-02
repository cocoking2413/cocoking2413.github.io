## 审批新建后台处理流程简要说明


#### 问题

1. 请求模型复杂多样，从而带来的安全、校正、验证等问题
2. 数据存储（子表/json）---parseJSON
3. 前端数据处理
4. 统计查询---内含子表：appListSub2


#### 基本流程

> 定义：  
> c:客户端   
> s:服务端UI层 
> bll:业务处理  

> ###### c=>s=>bll=>s=>c

> ###### requestModel=>checkModel=>parseModel=>updateModel=>insertModel=>resultInfo=>sendMsg=>returnInfo   

```
sequenceDiagram
loop create
c->>s:requestModel
s->>bll:checkModel
bll->>s:resultInfo<=insertModel<=updateModel<=parseModel
s->>c:resultInfo<=sendMsg
end
```
![img1](https://github.com/cocoking2413/cocoking2413.github.io/blob/master/static/img/2016-12-02_143935.png)


> ###### start=>c创建请求=>s检查校正数据(1)=>bll解析校正json(2)并转换成可用对象=>bll数据库新增审批记录流转记录（操作日志处理）=>s发送消息=>s返回结果=>c接收创建结果信息=>end

```
graph TD
A[start]-->B(c创建请求)
B-->C{s检查校正数据}
C-->|false|D[返回c]
C-->|true|E{bll解析校正json并转换成可用对象}
E-->|false|I[返回c]
E-->|true|F{bll数据库新增审批记录流转记录}
F-->|false|J[返回c]
F-->|true|G{s发送消息}
G-->|false|K[返回c]
G-->|true|H[end]
```
![img2](https://github.com/cocoking2413/cocoking2413.github.io/blob/master/static/img/2016-12-02_143803.png)


#### 几个点

1. 表单模型验证    

```csharp
	/// <summary>
	/// 表单基类
	/// </summary>
	public class FormBase : IValidatableObject {
		/// <summary>
		/// 继承类有特殊验证需要重写此方法
		/// </summary>
		/// <param name="validationContext"></param>
		/// <returns></returns>
		public virtual IEnumerable<ValidationResult> Validate(ValidationContext validationContext) {
			return new List<ValidationResult>();
		}
		/// <summary>
		/// 特性验证
		/// </summary>
		/// <returns>results未返回，需要返回再传回</returns>
		public virtual bool IsValid() {
			ValidationContext context = new ValidationContext(this);
			List<ValidationResult> results = new List<ValidationResult>();
			return Validator.TryValidateObject(this, context, results, true);
		}
	}
```


```csharp
        /// <summary>
		/// 验证int数组成员范围
		/// </summary>
		/// <param name="value"></param>
		/// <param name="validationContext"></param>
		/// <returns></returns>
		protected override ValidationResult IsValid(object value, ValidationContext validationContext) {
			try {
				int [ ] _value = value as int [ ];
				if ( _value.Length > 0 ) {
					foreach ( var item in _value ) {
						if (!base.IsValid(item) ) {
							return new ValidationResult("int array validate error!");
						}
					}
					return ValidationResult.Success;
				}
			}
			catch ( Exception ex ) { }
			if ( this.canNull ) return ValidationResult.Success; else return new ValidationResult("has int array validate inner error!");
		}


```

2. 多表单类型数据转换和验证
```csharp
		/// <summary>
		/// 数据序列化等及验证------主方法
		/// </summary>
		/// <typeparam name="T">继承自FromBase的表单类</typeparam>
		/// <param name="model">请求模型类</param>
		/// <param name="result">转换验证结果</param>
		/// <param name="approve">转换结果</param>
		/// <param name="OtherValidate">需要数据库验证的方法</param>
		/// <returns>转换结果</returns>
		private static oa_ApproveInfo MainFun<T>(CreatePostViewModel model, out TurnAndValidResult result, ref oa_ApproveInfo approve, Func<T,T> OtherValidate =null) where T : FormBase {
			try {
				var Form = JsonConvert.DeserializeObject<T>(model.JsonContent);//反序列化
				if ( OtherValidate != null ) Form=OtherValidate(Form);
				if ( Form!=null&& Form.IsValid()) {
					var validList = Form.Validate(new ValidationContext(Form));
					if ( validList == null || validList.Count() == 0 ) {
						result = TurnAndValidResult.Success;
						//重置jsonContent,确保格式无误
						model.JsonContent = JsonConvert.SerializeObject(Form);
						if ( !model.Turn(ref approve) )
							result = TurnAndValidResult.Error;
						return approve;
					}
				}
				result = TurnAndValidResult.ValidError;
				return approve;
			}
			catch ( Exception ex ) { }
			result = TurnAndValidResult.TurnError;
			return null;
		}
```

3. 数据统计
```sql
SELECT (select StringValue from dbo.parseJSON(@JSON) where name='Subject' and parent_ID=obj.Object_ID)
 as 'List.Subject', (select convert(decimal(18,2),StringValue) from dbo.parseJSON(@JSON) where name='Money' and parent_ID=obj.Object_ID)
 as 'List.Money', (select StringValue from dbo.parseJSON(@JSON) where name='Document' and parent_ID=obj.Object_ID)
 as 'List.Document', (select convert(int,StringValue) from dbo.parseJSON(@JSON) where name='Num' and parent_ID=obj.Object_ID)
 as 'List.Num', (select StringValue from dbo.parseJSON(@JSON) where name='Remark' and parent_ID=obj.Object_ID)
 as 'List.Remark'
from
 dbo.parseJSON(@JSON)
 obj
where
 obj.parent_ID=(select list.Object_ID from dbo.parseJSON(@JSON) list where NAME='List')
```