#Felxible Benefits M API Design
===================

##Content
[TOC]

##Web Api (Controller Api)


###AccountController
和用户的登录、密码相关的接口集合
####方法
名称     |说明         |类型
-------- | -----------|-----
SignIn(string, string)| 以用户名和密码作为输入登录结果|HttpPost
SSOSignIn()    | SSO方式登录|View
FindAccountWithEmailTicket(string)|以用户名或邮箱作为凭证，让系统发送找回密码邮件|HttpPost
FindAccount(string, string, string)|以用户名、用户姓名、证件号作为凭证，登录并重设密码|HttpPost
SetPassword(string) | 以邮件中的ticketId作为凭证，进入到找回密码页面 | View
SetPassword(string, string) | 以tieketId和新设定的密码来修改现在密码 | HttpPost
ResetPassword(string, string)|以旧密码为凭证，修改密码 | HttpPost


####**SignIn(string userId, string password)**
说明 HttpPost以用户名和密码作为输入登录结果
>输入 用户名和密码

>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult() { success = true, message = "" }
>例子 ApiCallResult() { success = false, message = "auth_failed" }


####**SSOSignIn()**
SSO方式登录，会根据HttpContext输入来验证，如果成功则跳转至第一菜单，如果不成功则跳转至 `SSOFail`页面


####**FindAccountWithEmailTicket(string info)**
以用户名或邮箱作为凭证，让系统发送找回密码邮件。

>输入 用户名或邮箱

>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(true, "email_reset_pwd", emailTo)
>例子 ApiCallResult(false, "user_not_found", "")

错误类型
同AccountService.FindAccountWithEmail
Code   | 说明
--------|--------
email_blank_fail_to_reset_account | 该用户没有电子邮箱
multi_user | 同时找到多个用户
user_not_found | 没有找到用户
unknown_exception | 未知异常


####**FindAccount(string userId, string userName, string nric)**
以用户名、用户姓名、证件号作为凭证，登录并重设密码
>输入 用户名、用户姓名和证件号

>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(){ success = true, message = "set_pwd_with_ticket", data=ticketId, action= {url = "../Sample/Account/SetPassword/" + ticketId} }
>例子 ApiCallResult(false, "user_check_failed", "")

错误类型
同SecurityService.FindAccountByMultiInfo
Code   | 说明
--------|--------
trs_user | TRS only用户无法使用此功能
user_check_failed | 用户与ticketId 校验失败
user_not_found | 没有找到用户
unknown_exception | 未知异常


####**SetPassword(string ticketId)**
用户通过找回密码的ticketId进入到的页面
>备注: 并不做ticketId校验
>输入 ticketId (Guid)


####**SetPassword(string ticketId, string password)**
根据ticketId及password设定新密码，并登录。
合并代替了以前的FirstResetPassword和Email找回密码功能

>输入 ticketId(Guid) 和新密码

>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(){ success = true, message = "set_pwd_with_ticket", data=ticketId, action= {url = "../Sample/Account/SetPassword/" + ticketId} }
>例子 ApiCallResult(false, "ticket_invalid ", "")

错误类型
同SecurityService.PasswordRegular, SecurityService.ChangePasswordAndUnlockByTicketAndSignIn

Code   | 说明
--------|--------
pwd_length_limit | 密码长度不符合要求
pwd_too_simple | 密码复杂度不符合要求
set_password_faot_found | 凭证未找到
user_not_found | 用户未找到
ticket_closed | 该凭证已使用
set_password_fail | 密码设定失败

####**ResetPassword(string oldpassword, string password)**
员工登录状态下，使用旧密码作为凭证，修改密码

>输入 老密码和新密码

>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(){ success = true, message = ""} }
>例子 ApiCallResult(false, "ticket_invalid ", "")

错误类型
同SecurityService.PasswordRegular, SecurityService.ChangePasswordByOldPassword

Code   | 说明
--------|--------
pwd_length_limit | 密码长度不符合要求
pwd_too_simple | 密码复杂度不符合要求
account_locked | 账户已锁
user_not_found | 用户未找到
auth_failed | 用户不在该公司内 / 密码不正确
set_password_fail | 密码设定失败





###BenefitController
和福利选择、展示相关的几个页面
####方法
名称     |说明         |类型
-------- | -----------|-----
GoBenefitSelect() | 以已保存的福利，并重新生成UI结构 | View
MyBenefit() | 已保存福利的页面 | View
BenefitSelect() | 福利选择页面 | View
SubmitBenefit(string) | 福利暂存 | HttpPost
SaveBenefit() | 福利保存 | HttpPost
SubmitSuccess() | 跳转页面 | View
HistoricalBenefit(int) | 历史已保存福利的页面 | View


####属性
名称     |说明        
-------- | -----------
MappingName | UI Product Mapping 表名


####内部方法
名称     |说明        
-------- | -----------
CreateCreditDictionary(Dictionary&lt;string, decimal>, List&lt;KeyValuePair &lt; string >>)  | 取得Wording并渲染 CreditDictionary方法


####**GoBenefitSelect()**
以已保存的福利，并重新生成UI结构。
一般这个方法用于即将跳转入福利选择页面之前。


####**MyBenefit()**
已保存福利的页面。展示当前以保存的福利及积分花费
#####ViewBag.Benefit 

>返回 [ApiCallResult](#ApiCallResult), 其data结构请参阅 List&lt;[CategoryView](#CategoryView)> SelectionService.GetSelection(string, string, string)
>例子 ApiCallResult(true, "", List&lt;[CategoryView](#CategoryView)>)

错误类型
Code   | 说明
--------|--------
exception | 未知异常
(该部分需要将异常分类更加具体化)

#####ViewBag.Credit
>返回 [ApiCallResult](#ApiCallResult)， 其data结构请参阅List&lt;[CreditDictonary](#CreditDictonary)> CreateCreditDictionary(Dictionary&lt;string, decimal>, List&lt;KeyValuePair&lt;string, string>>)
>例子 ApiCallResult(true, "", List&lt;CreditDictonary>);

错误类型
Code   | 说明
--------|--------
exception | 未知异常
(该部分需要将异常分类更加具体化)


####**BenefitSelect()**
福利选择页面，获取当前选择信息、积分信息、联动规则配置文件
#####ViewBag.Benefit 
(同上一段落 "MyBenefit")
#####ViewBag.Credit
(同上一段落 "MyBenefit")
#####ViewBag.AssociationRoles
>返回 [ApiCallResult](#ApiCallResult), 其data结构请参阅SelectionService.ReadProdConstraintConfig(string, string)
>例子 ApiCallResult(true, "", List<OptionConfig>)

错误类型
Code   | 说明
--------|--------
exception | 未知异常
直接异常 | 缺乏配置文件时直接异常


####**SubmitBenefit(string benefitStringData)**
福利暂存，将用户在页面上提交的数据暂存至UIProduct/UIOption
(在FlexM的设计中，这个步骤后直接进行SaveBenefit()福利保存)

>输入 List&lt;[SelectionPostItem](#SelectionPostItem)> 的Json序列化


>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(false, "post_item_convert_fail", ex.Message)

错误类型
Code   | 说明
--------|--------
post_item_convert_fail | 页面提交数据Json反序列化失败
uiproduct_convert_fail | UIProduct/UIOption转换失败
negativ_credit | 如果不允许负积分时，发现负积分
exception | 未知异常


####**SaveBenefit()**
福利保存，将UIProduct/UIOption数据保存

>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(true, "", "", new { url = "SubmitSuccess" } )
>例子 ApiCallResult(false, "exception", "", new { url = "BenefitSelect" })


####**SubmitSuccess()**
福利保存成功的跳转页面


####**HistoricalBenefit(int year)**
展示历史已保存福利的页面
#####ViewBag.Benefit

>输入 年份(或相关Code)

>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(true, "", List&lt;[CategoryView](#CategoryView)>)
>例子 ApiCallResult(false, "exception", ex.message)

#####ViewBag.Employee

>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(true, "", Dictonary&lt;string, string>)
>例子 ApiCallResult(false, "exception", ex.message)

#####ViewBag.Families

>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(true, "", List&lt;[DepListItem](#DepListItem)>)
>例子 ApiCallResult(false, "exception", ex.message)

#####ViewBag.Credit
(同上一段落 “MyBenefit”)

错误类型
Code   | 说明
--------|--------
no_selection_data | 没有福利数据
exception | 未知异常


###ContentController
不便于分类的功能集合
####方法
名称     |说明         |类型
-------- | -----------|-----
FileList(string) | 下载文件页面 | View
DownloadFile() | 下载文件 | HttpGet
FAQ() | 福利问答页面 | View

####**FileList(string errorCode)**
下载文件页面
>该页面的写法带有实验性，并不要推广

>返回 [ApiCallResult](#ApiCallResult)
>例子  new ApiCallResult(true, "", List&lt;[DownloadForm](#DownloadForm)>)

####**DowloadFile()**
下载文件，会直接返回文件流
>由于这个页面需要直接返回下载流，因而不能以JsonResult(ApiCallResult)形式返回

>错误 返回message
>例子 "file_missing"

错误类型
Code   | 说明
--------|--------
file_missing | 文件不存在
path_not_legal | 尝试访问非法路径
unknown_exception | 未知异常


####**FAQ()**
福利问答页面
>该页面的写法带有实验性，并不要推广

>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(true, "", List&lt;[BenefitOverview](#BenefitOverview)>)


###HomeController
####方法
名称     |说明         |类型
-------- | -----------|-----
Welcome() | 登录后的首页信息 | View

####**Welcome()**
登录后的首页信息
>该页面会自动将[[[EmployeeName]]]及[[[ClosingDate]]]替换成真实信息

>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(true, "", new { closingDate = closingDate, daysLeft = daysLeft, totalCredits = totalCredits })


###ProfileController
和个人信息、家属信息相关集合
####方法
名称      |说明        |类型
-------- | -----------|-----
MyInformation() | 个人/家属信息展示页面 | View
ConfirmInformation() | 编辑个人/家属信息页面 | View
UpdateInformation(string) | 更新个人信息 | HttpPost
AddDependent(string) | 添加家属 | HttpPost
SyncDependentList() | 获得家属 | HttpPost
GetDependent(string) | 根据ID获得家属信息 | HttpPost
DeleteDependent(string) | 根据ID删除家属 | HttpPost
UpdateDependent(string, string) | 更新家属信息 | HttpPost


####**MyInformation()**
个人/家属信息展示页面
#####ViewBag.employeeProfile
个人信息部分，从EmployeeService.GetFlexProfile(UserId, Lang)中获取数据
>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(true, "", List&lt;[EEItem](#EEItem)>)

错误类型
Code   | 说明
--------|--------
直接抛出异常 |可能为配置文件丢失

#####ViewBag.dependents
家属信息部分，从DependantService.GetDependants(UserId, Lang)中获取数据
>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(true, "", List&lt;[DepListItem](#DepListItem)>)

错误类型
Code   | 说明
--------|--------
直接抛出异常 |

####**ConfirmInformation()**
编辑个人信息/家属信息页面
#####ViewBag.employeeProfile
(同上一段落 “MyInformation”)
#####ViewBag.dependents
(同上一段落 “MyInformation”)
#####ViewBag.webcanadddep
是否能添加家属

>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(true, "", true)

错误类型
Code   | 说明
--------|--------
直接抛出异常 |

#####ViewBag.webgetdepconfig
读取家属配置，从DependantService.ReadDependantConfig(UserId, Lang) 获取数据
>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(true, "", List&lt;DepItem>)

####**UpdateInformation(string profileData)**
更新个人信息，从EmployeeService.SetFlexProfile(UserId, profileData.FromJson&lt;List&lt;[PostItem](#PostItem)>>())更新并返回结果

>输入 List&lt;[PostItem](#PostItem)> Json序列化数据

>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(true, "", "", new { url = "../Benefit/BenefitSelect" })
>例子 ApiCallResult(false, "unknown_exception", e.Message)

错误类型
Code   | 说明
--------|--------
unknown_exception |未知异常


####**AddDependent(string depData)**
添加家属从DependantService.AddDependant2(UserId, depData.FromJson&lt;List&lt;[PostItem](#PostItem)>>())添加并返回结果

>输入 List&lt;[PostItem](#PostItem)> Json序列化数据

>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(true, "", "")

错误类型
Code   | 说明
--------|--------
age_not_valid| 年龄超出范围，无法添加
parent_existed| 家属已存在
denpendent_has_same_name| 家属重名，无法添加
spouse_multiple | 无法添加多名配偶
children_multiple | 无法添加超出规定的子女
marrydate_is_empty | 配偶结婚日期不能为空
value_is_empty | 缺失必填值
dob_is_invalid  | 生日不合法
dom_is_invalid | 结婚日期不合法
nric_is_invalid | 证件号不合法
nric_is_invalid_role |  证件号不满足18位身份证自校验规则
nric_is_invalid_birthday |18位身份证与出生日期不匹配
nric_is_invalid_gender | 18位身份证与性别不匹配
dob_is_latter_then_join_date | 出生日期比入职日期晚，无法参加投保
spouse_marrieddate_notvalid |  iehun日期不合法
date_not_valid | 日期无法识别
data_truncated | 输入长度过长
nation_id_not_0_15_18 | 身份证并非15或18位
operation_failed | 未知错误
not_allowed_type | 无法添加不允许的家属类别
not_in_enroll | 注册期未开放，无法添加家属

####**SyncDependentList()**
获取员工家属列表。可根据公司设定和员工当前是否在注册期/LifeEvent内判断 是否可更改/删除信息。
从DependantService.GetDependants(employeeNo, lang)寻并返回结果

>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(true, "", List&lt;[DepListItem](#DepListItem)>)
>例子 ApiCallResult(false, "", e.Message)

####**GetDependent(string depId)**
获取家属详细信息
从DependantService.GetDependantDetail(UserId, depId, Lang)获取信息

>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(true, "", List&lt;[DepItem](#DepItem)>);
>例子 ApiCallResult(false, "unknown_exception", ex.Message)



####**DeleteDependent(string depId)**
通过DependantService.DeleteDependant2(UserId, depId)删除家属

>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(true, "", "");
>例子 ApiCallResult(false, "unknown_exception", ex.Message)

错误类型
Code   | 说明
--------|--------
delete_dependent_fail | 删除家属失败
cannot_delete_dependent | 用户不能删除该家属


####**UpdateDependent(string depData, string depId)**
更新家属信息
通过DependantService.UpdateDependant2(UserId, depId, depData.FromJson&lt;List&lt;[PostItem](#PostItem)>>())更新家属

>输入 List&lt;[PostItem](#PostItem)> Json序列化数据，和DependentId

>返回 [ApiCallResult](#ApiCallResult)
>例子 ApiCallResult(true);
>例子 ApiCallResult(false, "unknown_exception", ex.Message)


Code   | 说明
--------|--------
age_not_valid| 年龄超出范围，无法添加
parent_existed| 家属已存在
denpendent_has_same_name| 家属重名，无法添加
spouse_multiple | 无法添加多名配偶
children_multiple | 无法添加超出规定的子女
marrydate_is_empty | 配偶结婚日期不能为空
value_is_empty | 缺失必填值
dob_is_invalid  | 生日不合法
dom_is_invalid | 结婚日期不合法
nric_is_invalid | 证件号不合法
nric_is_invalid_role |  证件号不满足18位身份证自校验规则
nric_is_invalid_birthday |18位身份证与出生日期不匹配
nric_is_invalid_gender | 18位身份证与性别不匹配
dob_is_latter_then_join_date | 出生日期比入职日期晚，无法参加投保
spouse_marrieddate_notvalid |  iehun日期不合法
date_not_valid | 日期无法识别
data_truncated | 输入长度过长
nation_id_not_0_15_18 | 身份证并非15或18位
operation_failed | 未知错误
not_allowed_type | 无法添加不允许的家属类别
not_in_enroll | 注册期未开放，无法添加家属



## Service

## 重要Model


###**<span id="ApiCallResult">ApiCallResult</span>**
####含义
通用返回结构

####属性
名称    |  类型   |  含义  | 例子
-------|--------|--------|-------
success  | bool |  是否成功  | false
message| string |  返回信息  | "some_exception"
data   | object |  数据类型  | "BenefitsOverviewSection1"
action | object |  额外操作，一般是页面跳转 | {url = "../Sample/Account/SetPassword/" + ticketId} 

####构造函数
`ApiCallResult()`

`ApiCallResult(bool isSuccess, string resultMessage = "", object resultData = null, object targetAction = null)`


###**<span id="BannerMenu">BannerMenu</span>**
####含义
Banner总菜单

####属性
名称    |  类型   |  含义  | 例子
-------|--------|--------|-------
LeftItems  | List&lt;[BannerItem](#BannerItem)> |  左菜单  | 
RightItems | List&lt;[BannerItem](#BannerItem)> |  右菜单  | 
ActiveItemCode   | string |  当前Active项 (TBD) | ""



###**<span id="BannerItem">BannerItem</span>**
####含义
Banner分项菜单

####属性
名称    |  类型   |  含义  | 例子
-------|--------|--------|-------
Url    | string | 地址    | "/CJLR/Home/Welcome"
IconClassName | string | 菜单文字前的IconClass |  "glyphicon-pencil"
Target | string | 等于&lt;a页面跳转的 target | null
Name   | string | 显示的名字 | "首页"
Code   | string | Banner的Code | "Enroll"
ChildrenItems | List&lt;[BannerItem](#BannerItem)> | 子菜单项 | null






###**<span id="BenefitOverview">BenefitOverview</span>**
####含义
FAQ页面数据结构

####属性
名称    |  类型   |  含义  | 例子
-------|--------|--------|-------
title  | string |  FAQ文字标题  | "Understanding Your Plan"
content| string |  FAQ文字内容  | "Flexible benefit symbolizes a ..."
code   | string |  FAQ Code    | "BenefitsOverviewSection1"

###**<span id="CategoryView">CategoryView</span>**
####含义
福利选择页面、我的福利页面、历史福利页面的福利结构
结构为多层结构，其子层级依次为 ProductView, OptionView

####属性
名称     |  类型   |  含义   | 例子
--------|--------|---------|--------
Code    | string |  Category 分组 |  "IB"
Ordinal | int    |  Category 顺序 |  0
Name    | string |  Category 名称 |  "Risk Insurance"
Description|string| Category 描述 |  "Insurance coverage includes Accident Death &amp;  (AD&amp;D), ..."
Link    | string | 下载文件的Link | ""
TotalCredit |decimal| EE总花费积分 | 2312.50
TotalERCredit|decimal| ER总花费积分 | 0.00
TotalEEDeduction|decimal| EE工资扣减积分（适用于必须自付费的产品）|0.00
Products |List&lt;[ProductView](#ProductView)> | 产品列表 | 请点击这里>>


###**<span id="ProductView">ProductView</span>**
####属性
名称     |  类型   |  含义   | 例子
--------|--------|---------|--------
Code    | string |  Product Code |  "CJLR_GEN_IBSPO"
CategoryCode | string | Category 分组 | "IB"
Ordinal | int    |  Product 顺序 |  1
Name    | string |  Product 名称 |  "Risk Insurance (Spouse)"
Description|string| Product 描述 |  "Insurance coverage includes Accident Death &amp;  (AD&amp;D), ..."
Link    | string | 下载文件的Link | ""
InsuredName | string | 如果是员工，为""，如果是家属，为家属的姓名 | "张嘉盛"
InsuredId | string | 如果是员工，为""，如果是家属，为家属的Guid的字符串 | "7371554d-b642-469c-975e-9500e73c083a"
InsuredDependentType | string | 被保险人类型 | "Dependent"
Enabled | bool | 选项可选 | true
UIProductTemplateType | string | 产品模板 | "singleOption"
Clicked | bool | 产品是否被点击过 | true
OtherInfo1 | string | 附加信息1 | ""
OtherInfo2 | string | 附加信息2 | ""
OtherInfo3 | string | 附加信息3 | ""
TotalCredit |decimal| EE产品花费积分 | 230.00
TotalERCredit|decimal| ER产品花费积分 | 0.00
TotalEEDeduction|decimal| EE工资扣减积分（适用于必须自付费的产品）|0.00
Options |List&lt;[OptionView](#OptionView)> | 产品列表 | 请点击这里>>

####构造函数
####方法

###**<span id="OptionView">OptionView</span>**
####属性
名称     |  类型   |  含义   | 例子
--------|--------|---------|--------
Code    | string |  Product Code |  "CJLR_GEN_MDCHI"
Ordinal | int    |  Product 顺序 |  15
CoverageId | int | Option 序号 | 1 
Name    | string |  Product 名称 |  "Core Medical Insurance 20K (Company Fund)"
Description|string| Product 描述 |  "Covered Area: Mainland China ..."
Link    | string | 下载文件的Link | ""
Quantity | int | 份数 | 1
Enabled | bool | 选项可选 | true
Visiable |bool | 选项可见 | true
IsSelected | bool | 是否处于选中状态  | true
UnitCredit | decimal | Option EE单价  | 400.00
UnitERCredit | decimal | Option ER单价 | 2400.00
UnitEEDeduction | decimal | Option EE工资扣减积分  | 0.00
UnitEOIPending | decimal | Option EOI 暂扣部分积分 | 0.00
EOIStatus | string | EOI 状态Code | "ACTIVE"
EOIName | string | EOI 状态名称 | "已生效"
EOIHealthDeclarationDocumentType | string | TBD-健康告知书模板 | (TBD)
EOIHealthDeclarationDocument | string | TBD-健康告知书 | (TBD)
CoverageAmount | decimal | 保额 | 0.00
OptionRecordId | Guid | （系统）Option编号| "b1b06843-b474-400e-b2b7-27f0506acead"


####构造函数
####方法





###**<span id="CreditDictonary">CreditDictonary</span>**
####属性
名称     |  类型   |  含义   | 例子
--------|--------|---------|--------
Code    | string |  Product Code |  "total"
Name    | string | 名称 | "总额度"
Value   | decimal | 积分值 | 2770.00
Description | string | 补充说明 | "您的总额度将根据您2017年的实际服务天数进行折算。"
CssStyle | string | CSS Class | "text-primary"
FixedTo | int | 保留小数位数 | 2

####构造函数
`CreditDictonary(string code, string name, decimal value, string description = "", string cssStyle="text-primary", int fixedTo = 2)`


###**<span id="SelectionPostItem">SelectionPostItem</span>**
####属性
名称     |  类型   |  含义   | 例子
--------|--------|---------|--------
Code    | string |  Product Code |  "CJLR_GEN_MDCHI"
InsuredId | string | 如果是员工，为""，如果是家属，为家属的Guid的字符串 | ""
SelectedOptionOrdinal | int[] | 选中的选项Ordinal  | 1
SelectedOptionCoverageId | int[] | 选中的选项CoverageId  | 1
Quantity | int[] | 选中选项的份数 | 1
OtherInfo1 | string | 附加信息1 | ""
OtherInfo2 | string | 附加信息2 | ""
OtherInfo3 | string | 附加信息3 | ""
UnitCredit | decimal[] | 选项单价 | 240.00






##次要Model

###**<span id="DepListItem">DepListItem</span>**
家属列表的结构
####属性
名称     |  类型   |  含义   | 例子
--------|--------|---------|--------
ID      | string |  家属ID  | "c04ace42-7685-4009-b7ca-665766575ddf"
Name    | string |  家属姓名 | "王医菲"
Dob    | string | 家属出生日期string | "1982-08-28"
NRIC     | string | 证件号 | "310109198208280024"
CanEdit  | bool | 可编辑 | true
CanDelete | bool | 可删除 | true
CanSetCompanyPay | bool | 可以被设为公司付费 | false
Relationship | string | 关系Code |  "SPOUSE"
RelationName | string | 关系本地化文字 | "配偶"

###**<span id="DepItem">DepItem</span>**
####属性
名称     |  类型   |  含义   | 例子
--------|--------|---------|--------
Key   | string | Key | "relation"
Name | string | Key的本地化文字 | "关系"
Type | string | 输入类型(text, select ...) | "select"
Value | string | 值 | "SON"
Nullable | bool | 可为空 | false
MaxLength | int | 最大长度 | 20
DepType | string | 可选择的家属类别 | "SPOUSE SON DAUGHTER FATHER MOTHER"
SelectionItem | List&lt;SelectDepType> | 选项 | [{"key":"SON","value":"儿子"},{"key":"DAUGHTER","value":"女儿"}]
Message | string | 提示文字 | null
WarningMessage | string | 警告文字 | null

和[EEItem](#EEItem) 结构非常类似



####构造函数
####方法
###**<span id="DownloadForm">DownloadForm</span>**
####属性
名称     |  类型   |  含义   | 例子
--------|--------|---------|--------
code   | string | FileCode| "3_CN_GEN"
id | int | 实际主键| 2045
title | string | 下载文件的本地化标题 | "普通医疗保险风险提示"



###**<span id="EEItem">EEItem</span>**
####属性
名称     |  类型   |  含义   | 例子
--------|--------|---------|--------
Key   | string | Key | "name"
Name | string | Key的本地化文字 | "姓名"
Type | string | 输入类型 | "p"
Group | string | 属性分组 | null
Value | string | 值 | "武松松"
Nullable | bool | 可为空 | false
MaxLength | int | 最大长度 | 20
Message | string | 提示文字 | ""
WarningMessage | string | 警告文字 | ""

和[DepItem](#DepItem) 结构非常类似




###**<span id="PostItem">PostItem</span>**
####属性
名称     |  类型   |  含义   | 例子
--------|--------|---------|--------
name | string | key |
value | string | value |
