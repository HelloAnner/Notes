```shell
thread --all > /Users/anner/Downloads/1.txt
thread --state TIMED_WAITING -n 3
```

```shell
trace org.apache.commons.lang.StringUtils isBlank                                                                                                                                                          
trace *StringUtils isBlank                                                                                                                                                                                 
trace *StringUtils isBlank params[0].length==1                                                                                                                                                             
trace *StringUtils isBlank '#cost>100'                                                                                                                                                                     
trace -E org\\.apache\\.commons\\.lang\\.StringUtils isBlank                                                                                                                                               
trace -E com.test.ClassA|org.test.ClassB method1|method2|method3                                                                                                                                           
trace demo.MathGame run -n 5                                                                                                                                                                               
trace demo.MathGame run --skipJDKMethod false                                                                                                                                                              
trace javax.servlet.Filter * --exclude-class-pattern com.demo.TestFilter                                                                                                                                   
trace OuterClass$InnerClass *    
```

```shell
watch com.fr.decision.webservice.v10.login.LoginService login "{params,target,returnObj,throwExp}"


[arthas@96487]$ watch  com.fr.decision.webservice.v10.entry.ReportEntryService getReportTemplateTree "{params[0],returnObj}" 
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 426 ms, listenerId: 11
method=com.fr.decision.webservice.v10.entry.ReportEntryService.getReportTemplateTree location=AtExit
ts=2022-11-10 19:52:40; [cost=57.904166ms] result=@ArrayList[
@String[b5f0c2ee-640f-4039-a4d4-918b55354898],
@ArrayList[isEmpty=false;size=7],

[arthas@96487]$ watch -b com.fr.decision.webservice.v10.entry.ReportEntryService getReportTemplateTree params -x 2
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 412 ms, listenerId: 12
method=com.fr.decision.webservice.v10.entry.ReportEntryService.getReportTemplateTree location=AtEnter
ts=2022-11-10 19:54:31; [cost=0.037709ms] result=@Object[][
    @String[b5f0c2ee-640f-4039-a4d4-918b55354898],
    null,
    @FileExtension[][isEmpty=true;size=0],
]



[arthas@96487]$ watch -f com.fr.decision.webservice.v10.entry.ReportEntryService getReportTemplateTree returnObj
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 432 ms, listenerId: 13
method=com.fr.decision.webservice.v10.entry.ReportEntryService.getReportTemplateTree location=AtExit
ts=2022-11-10 19:55:07; [cost=58.927208ms] result=@ArrayList[
    @FileNodeBean[com.fr.decision.webservice.bean.entry.FileNodeBean@a0c0f63],
    @FileNodeBean[com.fr.decision.webservice.bean.entry.FileNodeBean@681b3902],
    @FileNodeBean[com.fr.decision.webservice.bean.entry.FileNodeBean@7c6fedf0],
    @FileNodeBean[com.fr.decision.webservice.bean.entry.FileNodeBean@3a220e1c],
    @FileNodeBean[com.fr.decision.webservice.bean.entry.FileNodeBean@50bf692c],
    @FileNodeBean[com.fr.decision.webservice.bean.entry.FileNodeBean@61dd8910],
    @FileNodeBean[com.fr.decision.webservice.bean.entry.FileNodeBean@62dee8b6],
```



```shell
stack org.apache.commons.lang.StringUtils isBlank                                                                                                                                                          
stack *StringUtils isBlank                                                                                                                                                                                 
stack *StringUtils isBlank params[0].length==1                                                                                                                                                             
stack *StringUtils isBlank '#cost>100'                                                                                                                                                                     
stack -E org\.apache\.commons\.lang\.StringUtils isBlank
```

