---
title: plantUML笔记
date: 2017-07-04 20:46:27
tags: [plantUML]
categories: [other]
---

plantUML是在markdown语法上实现绘制uml图的工具
### 绘制流程图的笔记
- 流程标签(activity label)以冒号开始，以分号结束,例如：
　:第一步;
  {% plantuml %}
  :第一步;
  {% endplantuml %}
- 使用关键字start和stop表示图示的开始和结束。
  @startuml
  start
  :Hello world;
  :This is on defined on
  several **lines**;
  stop
  @enduml

<!-- more --> 

{% plantuml %}
  @startuml
  start
  :Hello world;
  :This is on defined on
  several **lines**;
  stop
  @enduml
{% endplantuml %}
- 使用关键字if，then和else设置分支测试。标注文字则放在括号中.
@startuml
start
if (Graphviz installed?) then (yes)
:process all\ndiagrams;
else (no)
:process only
__sequence__ and __activity__ diagrams;
endif
stop
@enduml

{% plantuml %}
@startuml
start
if (Graphviz installed?) then (yes)
:process all\ndiagrams;
else (no)
:process only
__sequence__ and __activity__ diagrams;
endif
stop
@enduml
{% endplantuml %}
- 使用关键字repeat和repeatwhile进行重复循环
@startuml
start
repeat
:read data;
:generate diagrams;
repeat while (more data?)
stop
@enduml

{% plantuml %}
@startuml
start
repeat
:read data;
:generate diagrams;
repeat while (more data?)
stop
@enduml
{% endplantuml %}
- 使用关键字while和end while进行while循环
@startuml
start
while (data available?)
:read data;
:generate diagrams;
endwhile
stop
@enduml

{% plantuml %}
@startuml
start
while (data available?)
:read data;
:generate diagrams;
endwhile
stop
@enduml
{% endplantuml %}
- 还可以在关键字endwhile后添加标注，还有一种方式是使用关键字is。
@startuml
while (check filesize ?) is (not empty)
:read file;
endwhile (empty)
:close file;
@enduml

{% plantuml %}
@startuml
while (check filesize ?) is (not empty)
  :read file;
endwhile (empty)
:close file;
@enduml
{% endplantuml %}

- 使用关键字fork，fork again和end fork表示并行处理。
@startuml
start
if (multiprocessor?) then (yes)
fork
  :Treatment 1;
fork again
  :Treatment 2;
end fork
else (monoproc)
:Treatment 1;
:Treatment 2;
endif
@enduml

{% plantuml %}
@startuml
start
if (multiprocessor?) then (yes)
fork
  :Treatment 1;
fork again
  :Treatment 2;
end fork
else (monoproc)
:Treatment 1;
:Treatment 2;
endif
@enduml
{% endplantuml %}

- 注释note right|left xxxx  note end
@startuml
start
:foo1;
floating note left: This is a note
:foo2;
note right
This note is on several
//lines// and can
contain <b>HTML</b>
====
* Calling the method ""foo()"" is prohibited
end note
stop
@enduml

{% plantuml %}
@startuml
start
:foo1;
floating note left: This is a note
:foo2;
note right
This note is on several
//lines// and can
contain <b>HTML</b>
====
* Calling the method ""foo()"" is prohibited
end note
stop
@enduml
{% endplantuml %}

- 为活动(activity)指定一种颜色
@startuml
start
:starting progress;
#HotPink:reading configuration files
These files should edited at this point!;
#AAAAAA:ending of the process;
@enduml

{% plantuml %}
@startuml
start
:starting progress;
#HotPink:reading configuration files
These files should edited at this point!;
#AAAAAA:ending of the process;
@enduml
{% endplantuml %}

- 使用->标记，你可以给箭头添加文字或者修改箭头颜色
@startuml
:foo1;
-> You can put text on arrows;
if (test) then
-[#blue]->
:foo2;
-[#green,dashed]-> The text can
also be on several lines
and **very** long...;
:foo3;
else
-[#black,dotted]->
:foo4;
endif
-[#gray,bold]->
:foo5;
@enduml

{% plantuml %}
@startuml
:foo1;
-> You can put text on arrows;
if (test) then
  -[#blue]->
  :foo2;
  -[#green,dashed]-> The text can
  also be on several lines
  and **very** long...;
  :foo3;
else
  -[#black,dotted]->
  :foo4;
endif
-[#gray,bold]->
:foo5;
@enduml
{% endplantuml %}

- 使用管道符|来定义泳道,还可以改变泳道的颜色。
@startuml
|Swimlane1|
start
:foo1;
|#AntiqueWhite|Swimlane2|
:foo2;
:foo3;
|Swimlane1|
:foo4;
|Swimlane2|
:foo5;
stop
@enduml

{% plantuml %}
@startuml
|Swimlane1|
start
:foo1;
|#AntiqueWhite|Swimlane2|
:foo2;
:foo3;
|Swimlane1|
:foo4;
|Swimlane2|
:foo5;
stop
@enduml
{% endplantuml %}

### 一个完整的例子
@startuml
start
:ClickServlet.handleRequest();
:new page;
if (Page.onSecurityCheck) then (true)
:Page.onInit();
if (isForward?) then (no)
  :Process controls;
  if (continue processing?) then (no)
    stop
  endif
  if (isPost?) then (yes)
    :Page.onPost();
  else (no)
    :Page.onGet();
  endif
  :Page.onRender();
endif
else (false)
endif
if (do redirect?) then (yes)
:redirect process;
else
if (do forward?) then (yes)
  :Forward request;
else (no)
  :Render page template;
endif
endif
stop
@enduml

{% plantuml %}
@startuml
start
:ClickServlet.handleRequest();
:new page;
if (Page.onSecurityCheck) then (true)
:Page.onInit();
if (isForward?) then (no)
  :Process controls;
  if (continue processing?) then (no)
    stop
  endif
  if (isPost?) then (yes)
    :Page.onPost();
  else (no)
    :Page.onGet();
  endif
  :Page.onRender();
endif
else (false)
endif
if (do redirect?) then (yes)
:redirect process;
else
if (do forward?) then (yes)
  :Forward request;
else (no)
  :Render page template;
endif
endif
stop
@enduml
{% endplantuml %}
