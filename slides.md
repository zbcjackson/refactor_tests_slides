---
theme: seriph
background: https://source.unsplash.com/collection/94734566/1920x1080
class: 'text-center'
highlighter: prism
lineNumbers: false
drawings:
  persist: false
css: unocss
layout: cover
---

# 测试与重构

---
layout: intro
---

# 张博超

- 方润 - 程序员（Delphi/ASP.NET）
- Microsoft - SDET （C#）
- Irdeto - Lead Engineer / ScrumMaster（C#/javascript）
- 百度 - 敏捷教练
- Odd-e - 教练/顾问/培训师/程序员

<br/>
咨询/教练辅导涵盖产品研发各个方面：组织/流程/团队/工程能力等

部分客户：

- GE医疗Surgery - 大规模敏捷LeSS导入 (C++/Java)
- HSBC移动 - 工程实践教练辅导 (Java/Node.js/Javascript)
- 银科控股 - 敏捷教练辅导与导入 (Node.js)
- BMW - 大规模敏捷LeSS 导入 (C++/Python)
- 一加手机 - 大规模敏捷教练辅导与导入 (Java)
- ...

<img src="/me.jpg" class="absolute top-30 right-40 h-40 rounded shadow" />


<img src="/Specification By Example(zh).jpg" class="absolute bottom-30 right-40 h-30 rounded shadow" />
<img src="/The Art of Unit Testing(zh).jpg" class="absolute bottom-20 right-20 h-30 rounded shadow" />
<img src="/User Stories Applied(zh).jpg" class="absolute bottom-20 right-60 h-30 rounded shadow" />

---

# 回顾一下什么是重构

<v-click style="margin-top: 60px;">

- 🧑‍💻 **发现问题，减弱问题** - 识别代码臭味，尝试重构以降低该臭味
- 📝 **小步** - 控制每次引入的变化尽可能的小并且只干一件事情
- 🛠 **测试** - 尽可能频繁的运行测试以保证没有破坏代码原有行为
- 🛌 **日常** - 重构是日常行为，通常不应该是一个专门的任务

</v-click>


<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/guide/syntax#embedded-styles
-->

<style>
h1 {
  background-color: #2B90B6;
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

---

# 测试也需要重构

<v-click style="margin-top: 60px;">

- 测试的测试是什么？

</v-click>

<v-click style="margin-left: 20px;">

- 代码

</v-click>

<v-click>

- 重构测试的目的是什么？

</v-click>

<v-click style="margin-left: 20px;">

- 降低变更成本
- 提升可读性

</v-click>

---
layout: section
---

# Cucumber

---
layout: two-cols
---

# Feature

<div style="padding: 10px 20px;">

```gherkin
Feature: Calculator

  Scenario: Add
    When add 1 and 1
    Then get 2

  Scenario: Subtraction
    When subtract 1 from 2
    Then get 1
```

</div>

::right::

# Steps

<div style="padding: 10px 20px;">

```java
@When("add {int} and {int}")
public void add(int a, int b) {
    calculator.add(a, b);
}

@Then("get {int}")
public void get(int expected) {
    assertEquals(expected, calculator.getResult());
}

@When("subtract {int} from {int}")
public void subtract(int a, int b) {
    calculator.subtract(b, a);
}
```

</div>


---

# 啰嗦的文档

```gherkin
Feature: Login

  Background:
    Given there is a user with email "zbcjackson@odd-e.com" and password "password"

  Scenario: login_success
    When login with email "zbcjackson@odd-e.com" and password "password"
    Then login success

  Scenario: fail_incorrect_passwd
    When login with email "zbcjackson@odd-e.com" and password "incorrect-password"
    Then login fail with message "Email and password are invalid."

  Scenario: fail_nonexisting_email
    When login with email "zbcjackson@odd-e-dummy.com" and password "incorrect-password"
    Then login fail with message "Email and password are invalid."

  Scenario: fail_no_passwd
    When login with email "zbcjackson@odd-e-dummy.com"
    Then login fail with message "Password should not be empty"

  Scenario: fail_no_email
    When login with password "incorrect-password"
    Then login fail with message "Email should not be empty"
```

---

# 这样就好多了

<br/>

```gherkin
Feature: Login

  Background:
    Given there is a user with email "zbcjackson@odd-e.com" and password "password"

  Scenario: login_success
    When login with email "zbcjackson@odd-e.com" and password "password"
    Then login success

  Scenario Outline: Validation
    When login with email "<email>" and password "<password>"
    Then login fail with message "<message>"
    Examples:
      | email                      | password           | message                         | case           |
      | zbcjackson@odd-e.com       | incorrect-password | Email and password are invalid. | Wrong password |
      | zbcjackson@odd-e-dummy.com | incorrect-password | Email and password are invalid. | Wrong email    |
      | zbcjackson@odd-e-dummy.com |                    | Password should not be empty    | Empty password |
      |                            | incorrect-password | Email should not be empty       | Empty email    |
```

---

# 意外的细节

```gherkin
Feature: Account

  Background:
    Given there is a user with email "zbcjackson@odd-e.com" and password "password"

  Scenario: account_add_success
    When login with email "zbcjackson@odd-e.com" and password "password"
    Then login success
    When add account "wangbb" and balance "10"
    Then account "wangbb" and balance "10" add succeed

  Scenario: account_add_fail_balance_0
    When login with email "zbcjackson@odd-e.com" and password "password"
    Then login success
    When add account "wangbb" and balance "0"
    Then account "wangbb" and balance "0" failed with empty balance

  Scenario: account_add_fail_no_name_10
    When login with email "zbcjackson@odd-e.com" and password "password"
    Then login success
    When add balance "10"
    Then failed with "Name should not be empty"

  ...
```

---

# 用钩子来隐藏细节

```gherkin
@login
Feature: Account

  Scenario: account_add_success
    When add account "wangbb" and balance "10"
    Then account "wangbb" and balance "10" add succeed

  Scenario: account_add_fail_balance_0
    When add account "wangbb" and balance "0"
    Then account "wangbb" and balance "0" failed with empty balance

  Scenario: account_add_fail_no_name_10
    When add balance "10"
    Then failed with "Name should not be empty"

  ...
```

---

# 另一种细节：第三方类库的使用

<v-click style="margin-top: 60px;">

- 重复的使用细节
- 自己的接口
- 封装依赖
- 屏蔽API变更或者工具变更

</v-click>

---

# 操作细节：用户交互

<v-click style="margin-top: 60px;">

- 保留意图，屏蔽动作
- 屏蔽控件变化

</v-click>

---

# 页面样式

<v-click style="margin-top: 60px;">

- 屏蔽页面变化

</v-click>

---

# 架构分层

<style>
.block {
  border: 1px solid #CCC; 
  border-radius: 10px;
  background-color: aquamarine;
  color: black;
  width: 200px;
  height: 50px;
  text-align: center;
  padding-top: 15px;
  margin-top: 10px;
}
.dashed-block {
  border: 1px dashed #CCC; 
  border-radius: 10px;
  color: white;
  width: 200px;
  height: 50px;
  text-align: center;
  padding-top: 15px;
  margin-top: 10px;
}
</style>

<v-click style="margin-top: 60px;">
  <div class="block">业务逻辑 </div>
  <div class="block">工作流 </div>
  <div class="block">页面对象 </div>
  <div class="block">操作 </div>
  <div class="block">驱动 </div>
</v-click>

<div v-click style="margin-top: 1px;" class="absolute top-20 right-100">
  <div class="dashed-block">Feature </div>
  <div class="dashed-block">Steps </div>
  <div class="dashed-block">Page Object </div>
  <div class="dashed-block">Actions </div>
  <div class="dashed-block">Driver </div>
</div>

---
layout: section
---

# 单元测试

---

# 可读性第一

<v-click style="margin-top: 60px;">

- case名字
- AAA
- 屏蔽其他细节

</v-click>

---
layout: quote
---

# 消除重复：一切恶魔的根源

---
layout: section
---

# 鸡生蛋，蛋生鸡的问题

<!--
全局从最深的分支开始重构，从最浅的分支开始测试
局部非改了才可测，非测了才能改
-->

---
layout: quote
---

# 你依然还是依赖其它测试的

---

# 控制无测试的变更大小

<v-click style="margin-top: 60px;">

- 利用一些设计技巧
- 利用重构工具

</v-click>

---

# 创造接缝(Seam)

<v-click style="margin-top: 60px">

- 处理依赖
    - 隔离依赖
    - 注入依赖

</v-click>

<v-click>

- 处理多职责混杂长方法
    - 提取相对独立逻辑到独立类
    - 测试提取的独立类
    - 探寻独立业务概念

</v-click>

<div v-click class="absolute top-50 right-100">
  <div class="code-line short"></div>
  <div class="code-line"></div>
  <div class="code-line long"></div>
  <div class="code-line"></div>
  <div class="code-line short"></div>
  <div class="code-line tab-1"></div>
  <div class="highlight">
    <div class="code-line tab-1"></div>
    <div class="code-line tab-2"></div>
    <div class="code-line tab-2"></div>
    <div class="code-line tab-1"></div>
  </div>
  <div class="code-line short"></div>
</div>

<div v-click class="absolute top-50 right-20">

  ```java
    public class Seam {
    
    
        public void execute() {
        
        
        
        }
    }
    ```
  ```

</div>


  <div v-click class="highlight absolute top-70 right-30">
    <div class="code-line tab-1"></div>
    <div class="code-line tab-2"></div>
    <div class="code-line tab-2"></div>
    <div class="code-line tab-1"></div>
  </div>

<style>
.highlight {
  background-color: transparent;
  border: 1px solid red;
  border-radius: 10px;
  margin-top: 10px;
}
.code-line {
  background-color: orange;
  width: 100px;
  height: 1px;
  margin-top: 10px;
  margin-bottom: 10px;
}
.short {
  width: 50px;
}
.long {
  width: 200px;
}
.tab-1 {
  margin-left: 20px;
}
.tab-2 {
  margin-left: 40px;
}
</style>

---
layout: center
class: text-center
---

# 谢谢

wechat/twitter/gmail : zbcjackson
<div style="display: flex">

<img src="/wechat.png" class="h-30 rounded shadow" />
<img src="/Odd-e.png" class="h-30 rounded shadow" />

</div>
