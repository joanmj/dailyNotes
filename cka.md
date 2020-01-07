158
# 原有服务到microservice架构
* 重构方式：
  1. 停止新功能的实现，彻底重构
  2. 新功能用微服务方式实现，逐步重构；
* 挑战：
  * 重构方式
    * 用原语言实现（重用部分旧代码）更加经济；
    * 设计不好的应该以新技术架构思路重新设计实现
    * 与数据过分耦合的应用可能无法重构；
  * 重构完成后选择机制/工具，保持所有模块的弹性；
  *
# container Orchestration
1. Define the concept of container orchestration.
  container
  Microservices
  Container Orchestration
    已知的orchestrator ，推荐学习edx151x课程https://www.edx.org/course/introduction-cloud-infrastructure-linuxfoundationx-lfs151-x，

2. Explain the reasons for doing container orchestration.
3. Discuss different container orchestration options.
4. Discuss different container orchestration deployment options.

# Chapter 4. Kubernetes Architecture
* the Kubernetes architecture.
  * high Level arch
    * One or more master nodes
    * One or more worker nodes
    * Distributed key-value store, such as etcd.
![](assets/cka-431f4ba2.png)
  * detail level architecture
    * Master node
      manage state of a Kubernetes cluster;has the following components:
      * API server
        * kube-apiserver
      * Scheduler
      * Controller managers
      * etcd.





* Explain the different components for master and worker nodes.
* Discuss about cluster state management with etcd.
* Review the Kubernetes network setup requirements.
