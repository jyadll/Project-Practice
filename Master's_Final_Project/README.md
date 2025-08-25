# 项目简介

基于语法生成的智能PDF模糊测试系统设计与实现（硕士毕设）           2024.9-至今

项目角色：独立负责人

技术栈： Python | LLM | RAG | LlamaIndex | RAGAs | JSON Schema | PyMuPDF

项目背景：传统基于变异的模糊测试，在面对PDF这类具有复杂、严格语法的格式时，生成的无效测试用例过多，导致对核心逻辑的测试覆盖率极低，效率瓶颈凸显。

工作内容：设计并实现基于语法生成的智能模糊测试系统。核心目标是先构建一个高保真的PDF语法规则库，并以此为基础，驱动测试用例的自动化生成。历经3次重大方案迭代，确立了“RAG辅助知识提取 -> 人工验证与修正 -> 驱动语法引擎生成”的技术路线。应用RAG生成JSON Schema语法草案，再通过“AI草拟-专家验证”的流程，逐一校正字段与依赖关系，最终获得一个包含233条规则的高保真语法库，解决了LLM对底层语法认知不足的幻觉问题。

项目成果：
该设计与核心模块荣获2025年全国大学生软件创新大赛软件系统安全赛全国总决赛“作品赛”三等奖；
构建的语法库成功解决了语法生成测试的“冷启动”难题，也为领域知识库的自动化构建提供了新的思路。

# 项目结构
项目结构如下：
```cpp
/pdf_fuzzing_project/
├── knowledge_base_builder/    # 所有用于构建知识库的代码都在这里
│   ├── documents/                # 预处理后的PDF规范文档的文本嵌入向量
│   ├── pdf_schemas/              # 校正工作台
│   ├── rag_logs/                 # 中间结果，留着分析RAG的性能
│   ├── config.py                 # RAG配置
│   ├── discovery_system.py       # 构建知识库的主控代码
│   ├── generate_blueprint.py     # 与LLM对话获取语法信息
│   ├── get_tri_source_context.py # 提供3源上下文
│   ├── prompt.py
│   └── evaluation_logger.py      # 评估LLM生成的每个字段的分数
│
├── blueprints/                # 知识库的“成品”，所有233+份蓝图
├── fuzz_data/                 # 模糊测试的“原料”
│
├── generator.py               # 模糊测试引擎的核心部分
├── grammar.py                 # 模糊测试引擎的核心部分
├── fuzzer.py                  # 模糊测试引擎的主循环
└── CrashChecker.py            # 模糊测试引擎的辅助模块
```

# 项目架构与核心思路
## 项目架构
<img width="2048" height="1121" alt="image" src="https://github.com/user-attachments/assets/20b6091b-4008-4866-9935-a6a179c5e356" />
当前进行到语法生成引擎的设计

## 3源RAG系统的搭建
3源RAG系统的搭建以及人机协作校正字段的工作流程：
①知识库构建：
<img width="2048" height="874" alt="image" src="https://github.com/user-attachments/assets/9f1744e6-10af-4c1b-bd19-40ca2efee81e" />
②检索策略设计：
<img width="2048" height="950" alt="image" src="https://github.com/user-attachments/assets/5acb53f0-7799-4e22-a53c-e9d0e900c359" />
## 人机协作实现获取完备的PDF语法信息：
③顺藤摸瓜扩展子字段
<img width="2048" height="969" alt="image" src="https://github.com/user-attachments/assets/1ae158bf-ff53-424f-8857-dcb162f525d4" />
④最终获得了233条准确的PDF语法信息：
<img width="1022" height="326" alt="image" src="https://github.com/user-attachments/assets/9f6d5fbf-9d54-436d-9ba2-6b45816b4e81" />

## 语法生成引擎的设计
<img width="2048" height="1428" alt="image" src="https://github.com/user-attachments/assets/ffe3036a-bd48-454c-8c81-989def3bb3bc" />
为每个复杂的PDF类型设计了生成函数，基于RAG阶段获取的语法信息。
举例说明：
### string类型的生成

一个string类型的字段，通过设定的提示词，大语言模型会给出相应的生成方法，用"generation_method": "random_text",指明生成的string是随机生成的。示例如下：

```cpp
          {
            "type": "string",
            "generation_method": "random_text",
            "params": {
              "min_length": 1,
              "max_length": 100,
              "charset": "printable"
            }
          }
```

有如下几个类别的string：”uri”、”template_choice”、”from_file”等，这里”template_choice”是大模型在generate_hint里提供了一些模板选择，语法生成引擎直接选择模板就可以，”from_file”是从文件中读取数据嵌入到PDF文件中。

目前正在开发设计语法生成引擎。
