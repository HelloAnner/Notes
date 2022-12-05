
### 默认缓存配置

慢查询结果
```text
# Query_time: 0.016373  Lock_time: 0.000003 Rows_sent: 0  Rows_examined: 0
# Query_time: 0.012153  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
# Query_time: 0.018439  Lock_time: 0.000000 Rows_sent: 0  Rows_examined: 0
# Query_time: 0.015064  Lock_time: 0.000004 Rows_sent: 2039  Rows_examined: 4080
# Query_time: 0.028932  Lock_time: 0.000006 Rows_sent: 0  Rows_examined: 0
# Query_time: 0.017240  Lock_time: 0.000004 Rows_sent: 500  Rows_examined: 2042
# Query_time: 0.011253  Lock_time: 0.000004 Rows_sent: 500  Rows_examined: 2042
# Query_time: 0.015009  Lock_time: 0.000004 Rows_sent: 1  Rows_examined: 2042
# Query_time: 0.011141  Lock_time: 0.000003 Rows_sent: 0  Rows_examined: 2041
# Query_time: 0.023347  Lock_time: 0.000005 Rows_sent: 2039  Rows_examined: 2042
# Query_time: 0.013286  Lock_time: 0.001645 Rows_sent: 0  Rows_examined: 0
# Query_time: 0.014970  Lock_time: 0.000003 Rows_sent: 0  Rows_examined: 0
# Query_time: 0.012892  Lock_time: 0.000004 Rows_sent: 0  Rows_examined: 0
# Query_time: 0.021036  Lock_time: 0.000003 Rows_sent: 0  Rows_examined: 0
# Query_time: 0.023076  Lock_time: 0.000005 Rows_sent: 2040  Rows_examined: 2042
# Query_time: 0.012658  Lock_time: 0.000002 Rows_sent: 0  Rows_examined: 0
# Query_time: 0.016203  Lock_time: 0.000001 Rows_sent: 0  Rows_examined: 0
# Query_time: 0.022441  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
# Query_time: 0.017071  Lock_time: 0.000005 Rows_sent: 0  Rows_examined: 0
# Query_time: 0.011524  Lock_time: 0.000005 Rows_sent: 0  Rows_examined: 0
# Query_time: 0.013312  Lock_time: 0.000004 Rows_sent: 0  Rows_examined: 0
# Query_time: 0.012304  Lock_time: 0.000004 Rows_sent: 0  Rows_examined: 0
# Query_time: 0.010210  Lock_time: 0.000003 Rows_sent: 0  Rows_examined: 0
# Query_time: 0.011026  Lock_time: 0.000003 Rows_sent: 0  Rows_examined: 0
# Query_time: 0.016325  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
# Query_time: 0.027298  Lock_time: 0.000063 Rows_sent: 0  Rows_examined: 0
# Query_time: 0.011840  Lock_time: 0.000041 Rows_sent: 0  Rows_examined: 0
```

慢的线程堆栈:
```text
Affect(class count: 1 , method count: 1) cost in 434 ms, listenerId: 2
`---ts=2022-10-28 15:13:56;thread_name=http-nio-8090-exec-10;id=232;is_daemon=true;priority=5;TCCL=org.apache.catalina.loader.ParallelWebappClassLoader@65cf90b7
    `---[15675.060625ms] com.fr.decision.webservice.v10.authority.AuthorityService:getAuthorityEntityByCarrier()
        +---[0.03% 4.079459ms ] com.fr.decision.webservice.bean.authority.PrivilegeOperator:getFieldValueString() #130
        +---[0.05% 8.40975ms ] com.fr.decision.webservice.v10.security.SecurityService:checkXss() #130
        +---[0.00% 0.003042ms ] com.fr.stable.query.QueryFactory:create() #132
        +---[0.00% 0.002541ms ] com.fr.decision.webservice.bean.authority.PrivilegeOperator:getEntityIds() #132
        +---[0.00% 0.004667ms ] com.fr.stable.query.restriction.RestrictionFactory:in() #132
        +---[0.00% 0.002792ms ] com.fr.stable.query.condition.QueryCondition:addRestriction() #132
        +---[0.00% 0.002584ms ] com.fr.decision.webservice.bean.authority.PrivilegeOperator:getCarrierType() #133
        +---[0.00% 0.198209ms ] com.fr.decision.webservice.impl.privilege.RoleTypePrivilegeProcessFactory:getRoleTypePrivilegeProcessor() #133
        +---[0.00% 0.003625ms ] com.fr.decision.webservice.bean.authority.PrivilegeOperator:getEntityType() #134
        +---[0.00% 0.005334ms ] com.fr.decision.webservice.impl.privilege.RoleTypePrivilegeProcessFactory:getPrivilegeType() #134
        +---[0.00% 0.002542ms ] com.fr.decision.webservice.bean.authority.PrivilegeOperator:getPrivilegeTypes() #135
        +---[0.01% 0.972ms ] com.fr.decision.authority.base.constant.type.authority.AuthorityType:fromInteger() #135
        +---[0.00% 0.009ms ] com.fr.decision.webservice.impl.privilege.PrivilegeType:type() #136
        +---[0.00% 0.002958ms ] com.fr.decision.webservice.bean.authority.PrivilegeOperator:getCarrierId() #136
        +---[0.00% 0.002416ms ] com.fr.decision.webservice.bean.authority.PrivilegeOperator:getEntityType() #136
        `---[99.89% 15657.942125ms ] com.fr.decision.webservice.impl.privilege.RoleTypePrivilegeProcessor:getAuthorityEntity() #136

[arthas@18559]$ trace com.fr.decision.webservice.v10.authority.AuthorityService getAuthorityEntityByCarrier '#cost>200'
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 526 ms, listenerId: 3
`---ts=2022-10-28 15:17:38;thread_name=http-nio-8090-exec-9;id=231;is_daemon=true;priority=5;TCCL=org.apache.catalina.loader.ParallelWebappClassLoader@65cf90b7
    `---[9191.280375ms] com.fr.decision.webservice.v10.authority.AuthorityService:getAuthorityEntityByCarrier()
        +---[0.06% 5.244458ms ] com.fr.decision.webservice.bean.authority.PrivilegeOperator:getFieldValueString() #130
        +---[0.12% 10.799625ms ] com.fr.decision.webservice.v10.security.SecurityService:checkXss() #130
        +---[0.00% 0.002667ms ] com.fr.stable.query.QueryFactory:create() #132
        +---[0.00% 0.002458ms ] com.fr.decision.webservice.bean.authority.PrivilegeOperator:getEntityIds() #132
        +---[0.00% 0.053125ms ] com.fr.stable.query.restriction.RestrictionFactory:in() #132
        +---[0.00% 0.005708ms ] com.fr.stable.query.condition.QueryCondition:addRestriction() #132
        +---[0.00% 0.002083ms ] com.fr.decision.webservice.bean.authority.PrivilegeOperator:getCarrierType() #133
        +---[0.00% 0.005708ms ] com.fr.decision.webservice.impl.privilege.RoleTypePrivilegeProcessFactory:getRoleTypePrivilegeProcessor() #133
        +---[0.00% 0.001958ms ] com.fr.decision.webservice.bean.authority.PrivilegeOperator:getEntityType() #134
        +---[0.00% 0.003209ms ] com.fr.decision.webservice.impl.privilege.RoleTypePrivilegeProcessFactory:getPrivilegeType() #134
        +---[0.00% 0.001916ms ] com.fr.decision.webservice.bean.authority.PrivilegeOperator:getPrivilegeTypes() #135
        +---[0.03% 2.346333ms ] com.fr.decision.authority.base.constant.type.authority.AuthorityType:fromInteger() #135
        +---[0.00% 0.003ms ] com.fr.decision.webservice.impl.privilege.PrivilegeType:type() #136
        +---[0.00% 0.002041ms ] com.fr.decision.webservice.bean.authority.PrivilegeOperator:getCarrierId() #136
        +---[0.00% 0.00625ms ] com.fr.decision.webservice.bean.authority.PrivilegeOperator:getEntityType() #136
        `---[99.75% 9168.37925ms ] com.fr.decision.webservice.impl.privilege.RoleTypePrivilegeProcessor:getAuthorityEntity() #136
```


请求:
![[../../../attachments/Fine 权限缓存测试报告.png]]
![[../../../attachments/Fine 权限缓存测试报告-1.png]]

### 关闭二级缓存
![[../../../attachments/Fine 权限缓存测试报告-2.png]]
![[../../../attachments/Fine 权限缓存测试报告-3.png]]

### 仅关闭磁盘缓存

![[../../../attachments/Fine 权限缓存测试报告-4.png]]
![[../../../attachments/Fine 权限缓存测试报告-5.png]]
