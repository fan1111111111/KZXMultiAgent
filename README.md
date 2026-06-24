# chenguang-agent-

#### 介绍
《天宫医疗-智能体》集中于解决医疗领域的 Agent 业务构建。包括：
1. 智慧问诊 Agent
2. 报告解读 Agent
3. 药物 Agent
4. 知识文档 Agent
5. 运营数据 Agent

#### 软件架构
软件架构说明


#### 安装教程

```sh
# 创建环境
conda create -n tiangong python=3.13
# 激活环境
conda activate tiangong

# 安装依赖
pip install -r requirements.txt
```


## 指定端口启动
```
uvicorn src.main:app --port 8080 --reload
```

## env配置
把 .env.example 复制一份 叫 .env ，修改为自己的信息即可



之前导入的样例数据：
1. QA数据集： 药物（名字、作用、副作用、使用方法、注意事项、疾病）
2.  一个数据集喂三处：（以下三个是没聊天之前就建设好的）
3.    Neo4j/GraphDB/postgresql/AWS（图谱）： 医药三元组：药物、关系、疾病
4.    postgresql/MySQL（所有产品标配）：持久化存储详细信息  药物（名字、作用、副作用、使用方法、注意事项、疾病）
5.    Milvus/Elasticsearch/VectorDB/Chroma（搜索）：相似度搜索；    药物作用  
Agent = 模型 + 记忆 + 工具 + ReAct
这个是聊天的时候产生；【治理Agent的业务】长期记忆、短期记忆： 用于存储对话的历史信息。

所有中间件，啥时候需要用看业务。
智慧问诊，挂号（产生会诊记录以后）对接HIS系统：必须存。
药物Agent：查药物之间的关系。
CRUD是跟业务走的。





neo4j：图谱；多跳推理。       复杂关联关系搜索
postgresql：关系数据；（传统数据）： 精确搜索
Milvus：相似度搜索；         语义相似度（语料库的建设是必不可少的）



