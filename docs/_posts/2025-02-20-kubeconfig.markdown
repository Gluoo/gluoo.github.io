---
layout: page
title:  "Merge multiple kubeconfig files"
date:   2025-02-20 11:20:00 +0800
categories: jekyll update
---

1. 背景
有时候我们需要管理多个kubernetes cluster，可能会有多个kubeconfig文件，每次切换cluster需要手动更新KUBECONFIG环境变量，易错且不便捷

2. 方案
合并多个kubeconfig文件到一个文件
```bash
$ export KUBECONFIG=~/$CONFIG_DIR/oneconfig:~/$ANOTHER_DIR/anotherconfig 
$ kubectl config view --flatten > ~/.kube/config
```
3. 使用
查看contexts
```bash
$ oc config get-contexts
CURRENT   NAME           CLUSTER     AUTHINFO           NAMESPACE
          dev-cluster    clusterA    userA              testing
*         test-cluster   clusterB    userB              dev
```
切换context
```bash
$ oc config use-context dev-cluster
```