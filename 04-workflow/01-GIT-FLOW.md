# Git 流程


### 一、概述
 * [master] 为主分支，仅支持 [release] 分支合并请求
 * 规划分支、提交，以及分支线


### 二、环境

| slug  | 环境     | 分支                     | 说明            |
|-------|----------|--------------------------|-----------------|
| [local] | 本地环境 | origin/local/{author}  | [3.1. 开发阶段] |
| [dev]   | 开发环境 | origin/deploy/dev        | [3.2. 自测阶段] |
| [uat]   | 测试环境 | origin/deploy/uat        | [3.3. 测试阶段] |
| [beta]  | 灰度环境 | origin/deploy/beta       | [3.4. 线上验收] |
| [release] | 生产环境 | origin/release/{date}    | [3.5. 发布生产] |


### 三、流程

#### 3.1. 开发阶段
 * 开发人员从master check-out 各自的开发分支[local]；
 * 基于此分支开发需求 / 修复BUG，多人选择主负责人分支进行开发；
 * 按需求/BUG填写提交信息，[REQ|BUG-XXXX]{需求|BUG标题}-{A次数}；
   * [REQ-9527]用户信息增加头像编辑-A01
   * [REQ-9527]用户信息增加头像编辑-A02
   * [REQ-9527]用户信息增加头像编辑-A03
   * [BUG-9555]用户注册短信偶发失败-A01

#### 3.2. 自测阶段
 * 基于[dev]分支：cherry-pick 各提交；
 * [CI/CD] => 自动构建部署；

#### 3.3. 测试阶段
 * 基于[uat]分支：cherry-pick 各提交；
 * 修复测试反馈BUG的提交；
   * [REQ-9527]用户信息增加头像编辑-B01
   * [BUG-9555]用户注册短信偶发失败-B01
 * [CI/CD] => 自动构建部署；

#### 3.4. 线上验收
 * 从 [master] check-out [release]-{date} 分支
 * 基于[release]分支：规整提交记录；
 * 按需求逐个 cherry-pick 提交，将一个需求的多个提交折叠为一个提交；
   * [REQ-9527]用户信息增加头像编辑
   * [BUG-9555]用户注册短信偶发失败
 * 基于[beta]分支：cherry-pick 各需求提交；
 * 不用[release]替代[beta]的原因是，避免在规整时频繁触发构建；
 * [CI/CD] => 自动构建部署；
 
 
 

#### 3.5. 发布生产
 * 基于[release]分支：提交合并请求至 [master]；
 * [CI/CD] => 自动构建部署；

#### 3.6. 补充说明
 * [local]、[dev]、[uat]分支保留详细提交记录；
 * [release]、[beta]、[master]分支保持清晰的需求变更线；


### 四、附1：cherry-pick

```sh
git cherry-pick 254a7deb
git add -A
git commit -m "[REQ-9527]用户信息增加头像编辑-A03"
git push
```


### 五、附2：rebase

```sh
git log 取到要保留的最后一次commit 
git rebase -i ad133ec1
pick 改为 drop  -- 删除该commit
pick 改为 squash  -- 折叠该commit到上一commit
git push origin HEAD --force
```

