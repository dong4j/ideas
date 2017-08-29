# Jenkins 邮箱配置
1. 下载 Email Extension Plugin 插件
2. 系统管理 
    - 系统配置
    - Jenkins Location:
        - 系统管理员邮件地址: xxxxxxx@163.com




| 字段 | 示例  |  
| --- | --- |
| SMTP服务器: | smtp.163.com | 
|用户默认邮件后缀:|@163.com| 
|使用SMTP认证：|勾选|
|用户名:|你的邮箱 xxxx@163.com|
|密码:|password(163如果开了授权码,这里填授权码)|
|使用SSL协议:|勾选|
|Reply-To Address：|xxxxxxx@163.com|
|字符集:|utf-8|

在需要发送邮件的项目后配置：
增加构建后步骤——>Editable email Notification 
Project Recipient List ： 填写需要接收邮件的收件人，以逗号分割(eg: $DEFAULT_RECIPIENTS,lei.tang@venusource.com)
。。。（默认）
Attach Build Log        ：选择  Compress and Attach Build Log      (这样会发送日志附件)

在 Add Trigger 里，选择需要触发的策略(eg: Success 当构建成功后触发)

选择邮件接收者(eg: RecipientList 表示前面所填写的Project Recipient Lis)，具体点击后面问号查看

