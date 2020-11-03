exam-stack ![alt tag](https://api.travis-ci.org/phishman3579/java-algorithms-implementation.svg?branch=master)
==============================

培训考试系统（`Spring MVC`），由于开源框架时间间隔已久，新老技术迭代，因此，在原有的开源框架进行改进（升级jar、补全图片），保证系统正常运行。

> 原开源框架地址: https://github.com/imalexyang/ExamStack （感谢）

## Table of Contents
+ [Environment](https://github.com/loveisontheway/exam-stack#Environment)
+ [Project](https://github.com/loveisontheway/exam-stack#Project)
+ [Deploy](https://github.com/loveisontheway/exam-stack#Deploy)
+ [Explain](https://github.com/loveisontheway/exam-stack#Explain)
+ [FAQ](https://github.com/loveisontheway/exam-stack#FAQ)

## Environment
+ `JDK:` 1.8+
+ `Tomcat:` 8.x.x
+ `Spring MVC:` 5.3.0
+ `RabbitMQ:` 1.5.0.RELEASE
+ `httpclient:` 4.2.3

## Project
| name | description |
| :------ | :------ |
| common | 核心组件（实体类、工具类） |
| management | 后台考试管理系统 |
| portal | 学员培训考试系统 |
| scoreMarker | 通过RabbitMQ减小大量用户交卷导致的服务器故障几率 |
| MySQL | 版本要求5.0+ |
| RabbitMQ | 3.7.15（依赖于Erlang） |
| Erlang | 23.1 |

## Deploy

### 数据库

1. 在MySql中新建一个数据库`examstack`，字符集使用`utf8 -- UTF-8 Unicode`

1. 使用我们提供的`examstack.sql`还原`examstack`

1. 设置好对应的访问权限

1. 初始帐户
`username: admin/student`
`password: 123456/123456`

### 运行

1. 将`Management.war`和`Portal.war`放到Tomcat应用程序目录（`webapps`）下。
	
1. 启动Tomcat，webapps目录下会生成两个文件夹（`Management`和`Portal`）。
	
1. 分别进入到`Management/WEB-INF/Spring`和`Portal/WEB-INF/Spring`下修改`root-context.xml`文件,将数据库地址、用户名和密码修改成正确的内容。修改完成后重启tomcat服务器。

	*需要修改的内容如下:*
	
```xml
<property name="jdbcUrl" value="jdbc:mysql:/*.*.*.*:3306/examstack?useUnicode=true&amp;characterEncoding=UTF-8" />
<property name="user" value="root" />
<property name="password" value="***" />
```	

1. 访问`http://localhost:8080/Management`和`http://localhost:8080/Portal`可以进入到管理后台页面和学员页面，并可以正常登录，则应用配置成功。

	**注意**：*在完成这一步后学员考试交卷无法完成,需要部署`ScoreMarker`。*
	
1. 部署ScoreMarker
	
    `Linux下`
	解压scoreMarker到/opt/目录。
	确认config/scoremarker.properties文件配置正确。
	拷贝scoremarker 执行脚本到init.d目录下并检查脚本中的配置。

	`Windows下`
	解压scoreMarker到任意目录。
	确认config/scoremarker.properties文件配置正确。
	修改installService.bat中APP_HOME为scoreMarker目录。
    运行installService.bat后启动服务ScoreMarkerService服务。

## Explain

**系统架构**

1. 管理后台现在独立成一个新项目，不再和第一版一样和前台合在一起。
1. 引入RabbitMQ，用于接受用户提交的答题卡，通过ScoreMarker从消息队列获取答题卡并交卷，减小大量用户提交导致的服务器故障几率。
1. 试题内容存储格式由xml改为json
1. 增加教师角色，现在教师用户可以正确地使用自己的权限管理学员、试题、试卷、考试以及培训。
1. 优化系统界面，新的界面看起来更加清爽、专业。
1. 新增了DashBoard，管理界面看起来会更专业。
1. 练习历史现在专门用一张表记录，使开发相关统计变得更容易。

**考试和练习**

1. 考试现在分为正式考试和模拟考试两种，正式考试需要教师或管理员审核，而模拟考试不需要审核。正式考试又分为公有和私有两种类型，公有考试是可以申请的考试，私有考试则需要教师或管理员指定学员（这里由管理员指定也被我们认为是审核的一种方式）。
1. 新增审核功能，现在教师创建的试卷、考试都需要超级管理员审核。超级管理员自己创建的不需要审核。同时，超级管理员和教师也可以审核学员的考试申请。
1. 新增人工阅卷功能。包含主观题的考试试卷，教师或超级管理员通过人工阅卷后可以确定最终分数。全部是客观题的试卷不需要阅卷。
1. 新增考试成绩统计功能，可以查看特定考试下学员的分数，同时可以对分数进行排序。
1. 新增学习记录查询功能，教师和管理员现在可以方便地查看学员的练习记录、培训记录和考试记录。
1. 新增快速考试模式，通过输入准考证号即可直接进入到对应的考试页面。
1. 新增继续考试功能，现在学员在考试过程中中断考试后，继续进入考试后，学员的答题记录会恢复到中断前的状态。

**题库管理**

1. 优化试题修改功能，现在可以正确地修改试题的基本信息。

**其他**

1. 新增培训功能，教师或超级管理员可以发布培训资料（视频和pdf文档）。学员可以选择自己需要参加的培训进行学习，培训分为视频和pdf格式的文档两种。
1. 新增虚拟班级功能，教师或管理员现在可以通过虚拟班组很方便地管理学员。


## FAQ

1. **不能获得数据库连接**

	> Cause:org.springframework.jdbc.CannotGetJdbcConnectionException:Could not get JDBC Connection;nested exception is java.sql.SQLException:Access denied for user 'root'@'localhost'
	
	请检查数据库连接字符串是否正确，同时检查数据库名、用户名和密码是否设置正确。

1. **交卷失败**
	
	> `RabbitMQ`没有启动会导致应用程序连接`RabbitMQ`失败
	
	请检查`RabbitMQ`服务是否启动。

1. **学员交卷后，管理界面学员对应的状态没有改变**

	> 交卷成功后，学员考试状态会修改成`已交卷`或者`已阅卷`，如果在提示“交卷成功”后没有发生任何变化，证明ScoreMarker没有正常启动或者ScoreMarker调用接口失败。
	
	请检查ScoreMarker是否启动。同时请保证ScoreMarker能调用到Management提供的接口，这一点在ScoreMarker部署中已经说明。

1. **RabbitMQ、MySql、ScoreMarker无法启动或经常被Kill掉**

	> 我们在测试过程中发现，内存不足的情况下（我们使用的是1G内存），RabbitMQ、MySql、ScoreMarker经常被Kill，而且无法启动，查看日志会发现提示内存不够。
	
	查看下日志，如果是内存不够的原因，那就赶紧加内存吧。为了保证系统正常运行，内存不能低于2G。