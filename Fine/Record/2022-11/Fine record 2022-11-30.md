
和Cloud聊了一下service的依赖关系的问题

> 主要还是使用方申明依赖关系的问题

https://kms.fineres.com/pages/viewpage.action?pageId=597197376


https://work.fineres.com/browse/BI-117962
被升级问题打断，这里只要确认了升级顺序就不会存在问题


打磨了一下导出excel的问题，connection的初始化问题
https://code.fineres.com/users/anner/repos/plugin-decision-ldap-check-tool/commits/0b29dc177c5c9c4a060a9185917e5d1816402ee1




解决了一个无法启动assist的问题，还是需要更换一个jdk为amd的架构才可以，浪费性能啊


提交了两个行政需求 ： 更换社保位置 + 工作地点
https://www.jiandaoyun.com/workflow/process_instance/638707fe2b9d6b000a8ea976/task/638708a592622c000a8566b0?memberType=0&guestCorpId=

到现在才遇到arm的assist启动失败的问题，我有罪 hh


花了很多时间在assist上面


提交了excel的代码
https://code.fineres.com/projects/PG/repos/plugin-decision-ldap-check-tool/pull-requests/4/diff/

					
修复问题 
https://work.fineres.com/browse/BI-117989


和Cloud 开了一个会
- IOC 反向注册context上下文概念 是一个比较好的想法
- Not Thread Safe 注解 - 一些有趣的注解 

修复注入导致的第二个问题
https://code.fineres.com/projects/DEC/repos/decision/pull-requests/11967/overview


一个tearDown问题卡了一个下午 shit
开始搞单元测试吧