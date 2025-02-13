---
sidebar_position: 2
---

# 委托访问
用户把访问自己资源的这件事委托给应用去做，让应用可以有限地访问用户资源。 

当用户尝试登录应用并使用应用去访问一些资源的时候（例如用应用打开自己的文件用于打印），应用需要首先请求获取资源的委托访问权限，以便代表用户访问资源。这种常见的场景被称为委托访问。


|                              委托访问                              |                            用访问凭证实现委托访问                             |
|:--------------------------------------------------------------:|:------------------------------------------------------------------:|
| ![委托访问简图](/img/learn/permission/delegated_access_overview.png) | ![委托访问凭证访问](/img/learn/permission/delegated_access_with_token.png) |


:::info Note
在使用委托访问时，你的应用永远无法访问用户本身就无法访问的资源。
:::

## 为什么要使用委托访问？
* 人们常常使用应用去访问自己的资源。例如，有人想要用一个PDF应用来查看他在钉盘上的文件。再比如，一个公司的业务线应用想检索有关同事的共享信息，以便轻松地为审批选择审批人。在这些场景中，应用都需要向当前登录应用的用户获取访问他们数据的权限。 
* 每当你想的应用想访问当前登录用户的资源时，就可以使用委托访问。无论是用户在收件箱中删除电子邮件，还是管理员为整个组织设置策略，所有涉及用户操作的场景都应该使用委托访问。 
* 相反，对于在没有特定登录用户的场景下，委托访问就不是一个好的选择了。例如你的应用涉及访问许多用户资源的操作，如备份、批量上传作业等自动化场景。对于这些类型的操作，你应该使用[应用访问](/docs/learn/permission/intro/application_permission)。

## 作为一个应用向用户申请权限
* 首先，开发者需要在开发者后台为应用勾选所需的委托权限项。
* 其次，你的应用需要向用户发起申请，让用户为你的应用需要访问的资源授予一个或一组特定的权限项。这些权限项描述了你的应用想要代表用户去执行的资源和操作。比如，你希望你的应用展示一个用户当前的日程，你需要请求用户同意授予权限项 `Calendar.Event.Read` 。 
* 一旦你的应用请求了权限项，正在使用应用的用户或者管理员就需要决定去是否授予权限。如果是以个人账号使用钉钉的用户（没有加入或切到任何组织），总是为自己授权给应用。如果是以组织账号使用钉钉（切到某组织），那么能否授权取决于权限项的属性。如果用户不能直接同意权限项，那么可以通知组织管理员去为用户同意授权。
* **请遵循最小权限原则**：永远不要向用户请求你的应用用不到的权限项。例如，如果你的应用想要列出用户的聊天列表而不是每一条聊天消息，你需要请求获取 `Chat.ReadBasic` 而不是 `Chat.Read` 。
  * 这可以帮助你的应用在被攻陷时降低安全风险，也可以帮助你的应用更容易地通过授权同意。
  * 如果用户怀疑你的应用存在超量获取权限的行为，平台会将收到的用户反馈同步给你的应用。

## 委托权限示例——通过DingTalk Graph访问钉盘
例如：小明想要用某应用打开一个钉盘中的文件。
* 对于用户本身的权限，钉盘服务会校验文件是否保存在小明的钉盘里。如果文件储存在别人的文档里，那么钉盘会用“无权限”这一理由拒绝小明的请求，除非小明有查看别人的钉盘的权利。
* 对于应用的委托访问，钉盘会校验应用是否被当前登录用户授予了 `Files.Read` 权限项。在这个例子中，当前登录用户是小明。如果 `Files.Read` 没有被小明授予给该应用，那么钉盘会拒绝请求。

| GET /drives/{id}/files/ | 应用被小明授予了 `Files.Read`                                                   | 应用没有被小明授予 `Files.Read`                              |
|-------------------------|-------------------------------------------------------------------------|-----------------------------------------------------|
| 文件在小明的钉盘里               | 200 - 允许访问。                                                             | 403 - 无权限。小明（或者他的主管）不允许应用读取他的文件。                    |
| 文件在别人的钉盘里               | 403 - 无权限。小明没有权限去读取别人的文件。即便是应用被小明授予了Files.Read，但由于小明没有读取别人文件的权利，还是返回失败。 | 403 - 无权限。小明没有权限读取该文件，而且小明也没有授予应用 `Files.Read` 的权限。 |
这里通过简单的例子来说明委托访问。现实的情况会更加丰富，比如小红是小明的主管，小红本来就可以访问小明的文档，那么在小红把委托权限授予应用后，小红可以用应用打开小明的文档。但是不变的原则是，你的应用永远无法访问授权用户本身无法访问的资源。更多钉盘的场景，详见钉盘文档。
