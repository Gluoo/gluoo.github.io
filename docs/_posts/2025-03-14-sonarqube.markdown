---
layout: page
title:  "Sonarqube"
date:   2025-03-14 16:40:00 +0800
categories: jekyll update
---

# Sonarqube入门

## 1. 什么是 SonarQube？

SonarQube 是一个开源的代码质量管理和静态代码分析工具。支持多种编程语言，包括 Java、JavaScript、Python、Go 等。可检测代码中的漏洞、安全问题、不规范代码、重复代码等。提供详细的代码质量报告，帮助开发团队提升代码质量。

![sonarqube](/images/sonar-development-workflow.webp)

## 2. SonarQube 的核心功能

静态代码分析：自动扫描代码，检测语法错误、不规范代码、漏洞。

代码质量指标：技术债务、复杂度、重复率、覆盖率等。

安全扫描：检测 OWASP Top 10、CWE 漏洞。

CI/CD 集成：支持 Tekton, Jenkins、GitLab CI、Azure DevOps、GitHub Actions 等。

规则引擎：基于自定义或内置规则检查代码质量。

![sonarqube-sample](/images/sonar-scan-result.png)

## 3. SonarQube 体系结构

- Scanner（扫描器）：从代码仓库提取代码并进行静态分析，并将结果上传给server。

[下载链接](https://docs.sonarsource.com/sonarqube-server/latest/analyzing-source-code/scanners/sonarscanner/)

使用示例：
在源代码根目录，创建一个`sonar-project.properties`

```
# must be unique in a given SonarQube Server instance
sonar.projectKey=my:project

# --- optional properties ---

# defaults to project key
#sonar.projectName=My project
# defaults to 'not provided'
#sonar.projectVersion=1.0
 
# Path is relative to the sonar-project.properties file. Defaults to .
#sonar.sources=.
 
# Encoding of the source code. Default is default system encoding
#sonar.sourceEncoding=UTF-8
```

运行`sonar-scanner`, scanner就开始分析代码，并把结果上传到server。


- Server（服务器）：处理扫描结果，提供 Web UI 界面。
![sonarqube-server](/images/sonar-server.png)

- Database（数据库）：存储分析数据、质量报告。

```bash
$ 🚀  kubectl get po -n sonarqube
NAME                         READY   STATUS    RESTARTS   AGE
sonarqube-8457f4dbf6-rtdwz   1/1     Running   0          44h
sonarqube-postgresql-0       1/1     Running   0          46h
```

社区版sonarqube，连接的是postgresql数据库，当前数据库实例为`sonarqube-postgresql-0`

SonarQube 的代码质量分析数据、报告、配置等存储在 PostgreSQL（或其他 RDBMS）中，主要涉及以下几个核心表：

- 代码分析数据
  这些表存储 SonarQube 的扫描结果，包括问题、漏洞等。

  - issues —— 存储代码问题（Bugs、Vulnerabilities、Code Smells），包含 severity（严重级别）、status（状态）、message（问题描述）等字段。
  - snapshots —— 存储每次分析的快照（即分析历史记录），对应每个 project 的版本。
  - measures —— 存储各种代码质量度量指标，如 代码行数、重复代码、覆盖率 等。

- 项目和分支
  - projects —— 存储项目的基本信息，如 name、key、uuid。
  - branches —— 存储项目的分支信息，用于管理多分支分析结果。

- 质量分析结果
  - components —— 存储代码的层次结构（如 项目、模块、文件）。
  - rules —— 存储 SonarQube 规则（用于静态代码分析）。
  - quality_gates —— 存储质量门槛配置（Quality Gates）。
  - quality_gate_conditions —— 存储每个质量门槛的具体条件，例如 代码覆盖率 > 80%。

- 运行分析的任务
  - ce_activity —— 存储分析任务（Background Task），用于跟踪扫描状态和结果。
  - ce_queue —— 处理中的分析任务队列。

查询示例：
```bash
$ kubectl exec -it sonarqube-postgresql-0 -- bash
I have no name!@sonarqube-postgresql-0:/$ psql -U bn_sonarqube -d bitnami_sonarqube
Password for user bn_sonarqube:
psql (17.4)
Type "help" for help.

bitnami_sonarqube=> select * from issues;
                 kee                  |              rule_uuid               | severity | manual_severity |

  message
                   | line |           gap           |  status   | resolution |             checksum             | a
ssignee |       author_login        | effort |  created_at   |  updated_at   | issue_creation_date | issue_update_d
ate | issue_close_date |             tags             |            component_uuid            |             project_
uuid             |

                                                                              locations

                                 | issue_type | quick_fix_available | rule_description_context_key | message_format
tings | code_variants | clean_code_attribute | prioritized_rule
--------------------------------------+--------------------------------------+----------+-----------------+--------
-------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------
-------------------+------+-------------------------+-----------+------------+----------------------------------+--
--------+---------------------------+--------+---------------+---------------+---------------------+---------------
----+------------------+------------------------------+--------------------------------------+---------------------
 474e81d7-6ae7-474f-b663-883a0368c92b | e55c7885-891b-40ae-a818-30d1c0a9861b | MINOR    | f               | Remove
this unused import of 'useMemo'.

                   |    2 |                         | OPEN      |            | 72ea64d94f6668974e0987bb9fa46b2c |
        | qian.du@ctigroup.hk       |      1 | 1741934877685 | 1741934877685 |       1740921491000 |     1741934801
000 |                  | es2015,type-dependent,unused | aab4e80a-441c-4d0f-949b-f2cb0f66038c | 5d2a2759-4120-4d4b-9
814-9007dfb877f6 | \x0a0808021002181d20241a203836303030393535623434653466383634306132316264373761666636616136
```


- Plugins（插件）：扩展功能，如支持不同语言、规则集、CI/CD 集成等。

![sonarqube-plugin](/images/sonar-plugin.png)

## 4. 如何使用 SonarQube？

### 4.1 安装与部署SonarQube Server

- Prerequiestes:
  - Kubernetes: 1.23+
  - Helm: 3.8.0+
  - Loadbalancer，charts默认使用Loadbalancer类型的service对外暴露服务

- Option 1: 官网提供的charts，https://artifacthub.io/packages/helm/sonarqube/sonarqube， 试了N多次，社区版本根本装不上，各种问题一个接一个。为啥叫社区版，免费但是坑得自己填。

- Option 2: 第三方提供的charts，bitami提供的charts还行，https://github.com/bitnami/charts/tree/main/bitnami/sonarqube。 按照guide，安装起来还算顺利，sonarqube server pod可能需要重启几次，启动需要花费几分钟到十几分钟，没特殊问题的话，自己重启几次就可以了。

但是安装完portal始终无法登录，不管是默认用户名密码，还是在chart里设置的，都无法登录，气到怒砸键盘，网上各种解决方案，无一靠谱，最后还是看官方文档，尝试直接去db里重置下密码，搞定。

进到postgresql pod，重置admin密码
```bash
$ kubectl exec -it $(kubectl get po -l app.kubernetes.io/name=postgresql --no-headers | awk '{print $1}') -- bash
  psql -U bn_sonarqube -d bitnami_sonarqube

  update users set
  crypted_password='100000$t2h8AtNs1AlCHuLobDjHQTn9XppwTIx88UjqUm4s8RsfTuXQHSd/fpFexAnewwPsO6jGFQUv/24DnO55hY6Xew==',
  salt='k9x9eN127/3e/hf38iNiKwVfaVk=',
  hash_method='PBKDF2',
  reset_password='true',
  user_local='true',
  active='true'
  where login='admin';

```
这就把密码重置为`admin`, 然后再登录portal就没问题了

### 4.2 配置 SonarQube

登录到portal， 创建项目，获取 SonarQube Token。

配置 SonarQube Scanner，在 sonar-project.properties 添加：

sonar.projectKey=my_project
sonar.host.url=http://$server_ip:$server_port
sonar.login=my_token

运行扫描命令：

sonar-scanner

在 SonarQube Web UI 查看代码分析结果。


## 5. SonarQube 在 CI/CD 中的应用

创建SonarQube Task
```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: sonarqube-scanner
spec:
  workspaces:
    - name: source
      description: "包含源代码的工作空间"
  steps:
    - name: run-sonarqube-scan
      image: sonarsource/sonar-scanner-cli:latest
      workingDir: $(workspaces.source.path)
      script: |
        #!/usr/bin/env bash
        sonar-scanner
```

创建一个pipeline
```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: build-and-scan-pipeline
spec:
  workspaces:
    - name: shared-workspace
  tasks:
    - name: git-clone
      taskRef:
        name: git-clone
      workspaces:
        - name: output
          workspace: shared-workspace
      params:
        - name: url
          value: "https://github.com/your/repo.git"
        - name: revision
          value: "main"
    - name: sonar-scan
      taskRef:
        name: sonarqube-scanner
      runAfter:
        - git-clone
      workspaces:
        - name: source
          workspace: shared-workspace
```


## 6. SonarQube 规则和质量门槛

质量门槛（Quality Gate）：设定代码质量标准，如：

代码覆盖率 > 80%

不能有 Blocker 级别的漏洞

代码重复率 < 10%

违反质量门槛时，CI/CD 失败，阻止代码合并。

### 配置 SonarQube Quality Gate

创建或编辑 Quality Gate   
登录 SonarQube：   
以管理员身份登录 SonarQube（默认用户 admin）。   
进入 Quality Gates 配置：   
点击左侧菜单 “Quality Gates”，然后点击 “Create” 创建新门槛，或选择现有门槛（例如默认的 “Sonar way”）进行编辑。   

设置条件：   
点击 “Add Condition”，添加以下规则：

代码覆盖率 > 80%：   
选择指标：Coverage   
操作符：Less than   
错误阈值：80%   
范围：On Overall Code（或根据需要选择 On New Code）   

无 Blocker 级别漏洞：   
选择指标：Blocker Issues   
操作符：Greater than   
错误阈值：0   
范围：On Overall Code   

代码重复率 < 10%：   
选择指标：Duplicated Lines (%)   
操作符：Greater than   
错误阈值：10%   
范围：On Overall Code   

保存设置：
命名你的 Quality Gate（例如 TeamQualityGate），并保存。   

应用到项目：   
在 “Projects” 页面，选择你的项目，点击 “Project Settings” > “Quality Gate”，选择刚创建的 TeamQualityGate。   

### 验证 Quality Gate
运行一次代码扫描（sonar-scanner），然后在 SonarQube 项目仪表盘查看 Quality Gate 状态：   
Passed：绿色，表示通过。     
Failed：红色，表示未通过，显示具体失败的条件。    

## 7. SonarQube 许可证

Community Edition（社区版）：免费，支持主要功能。

Developer Edition（开发者版）：支持更多语言、分支分析。

Enterprise Edition（企业版）：支持高级安全分析、依赖管理。

Data Center Edition（数据中心版）：适用于大规模分布式环境。

## 8. 总结

SonarQube 是一个强大的代码质量管理工具。

通过静态分析、CI/CD 集成，提高代码安全性和可维护性。

可帮助团队减少技术债务，提升开发效率。



