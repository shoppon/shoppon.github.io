---
title: "C++单元测试"
date: 2020-07-17T19:26:38+08:00
categories: ["c++"]
draft: true
---

# C++单元测试

## 测试目的

新增、修改代码时更有信心，提高效率，更好享受人生。好的单元测试会反哺设计，单元测试不好做很多时候意味着架构设计不够好。

## 测试范围

1. 正常功能。非编译型语言中还可保证语法正确。
2. 边界值。
3. 异常分支。

## 测试时机

1. 写代码之前。**核心逻辑**先写测试用例，再写业务逻辑，可以用测试用例调试业务代码。
2. 发现Bug之后。发现Bug之后，举一反三，通过测试用例防护各种异常场景。
3. 不求面面俱到。简单的逻辑没有必要写过多的单元测试，边际成本太高。

## 测试要点

- 多线程
  - 依赖队列，依赖select/epoll等等。
- socket相关
  - 不适合单元测试，通过自动化集成测试验证。
- 大对象
  - 审视架构、设计是否合理。

## 测试划分

- 多级调用时，底层详细测试，高层主要流程。
- 单元测试针对函数、模块级即可。

## gtest使用

### 测试集

通过继承`::testing::Test`来构造测试集：

```c++
class S3HandlerTest : public ::testing::Test {
protected:
    virtual void SetUp();
}
```

然后通过`TEST_F`宏构造测试用例：

```c++
TEST_F(S3TomaTest, SendDatasetStart)
{
}
```

`TEST_F`本质上创建一个**新类继承**测试集，因此要注意测试集中函数和变量的访问权限。

### 断言

#### ASSERT_EQ

通过`ASSERT_EQ`宏来简单判断结果是否符合预期：

```c++
ASSERT_EQ(DPP_MAGIC, elem0->magic);
ASSERT_EQ(DPP_DATASET_START, elem0->cmd_type);
```

### Mock

#### Mock外部依赖

程序中有时局部变量会依赖外部资源，在LLT时是不可达的：

```c++
DatasetSeq S3Handler::PullDatasetSeq()
{
    DatasetSeq seq;
    // 必须一次性获取所有dataset文件和done文件，分开获取可能导致数据不一致
    std::string prefix = m_sid.string() + "/";
    S3Connection s3Conn = S3Connection::GetInstance(m_s3BucketCtxt);
    std::vector<ObjectContent> objects = s3Conn.ListObjects(prefix.c_str(), nullptr, CMD_PREFIX.c_str()).first;
}
```

首先需要将外部依赖`s3Conn`提取成函数，然后通过继承来mock

```c++
virtual shared_ptr<S3Connection> GetConnection();

class MockS3Connection : public S3Connection {
public:
    MockS3Connection(S3BucketCtxt& ctx) : S3Connection(ctx){};
    ~MockS3Connection(){};

    MOCK_METHOD4(ListObjects, Pairvb(const char* prefix, const char* marker, const char* delimiter, int maxkeys));
    MOCK_METHOD3(Read, ssize_t(const std::string& objKey, char* buf, const size_t bufSize));
};

class MockS3Handler : public S3Handler {
public:
    MockS3Handler(Reactor& r, std::shared_ptr<TaskPool> pool, std::shared_ptr<TaskPoolHandler> handler,
        std::shared_ptr<TaskPool> fileOpsPool, std::shared_ptr<TaskPoolHandler> fileOpsPoolHandler)
        : S3Handler(r, pool, handler, fileOpsPool, fileOpsPoolHandler){};

    ~MockS3Handler(){};

    MOCK_METHOD0(GetConnection, shared_ptr<S3Connection>());
};
```

#### 定义mock函数行为

通过`EXPECT_ALL`来定义mock函数的行为：**调用次数**、**返回值**、**保存参数**等。

```c++
Pairvb pv = {m_objs, true};
EXPECT_CALL(*(m_s3Conn.get()), ListObjects(testing::_, testing::_, testing::_, testing::_)).WillOnce(Return(ByMove(pv)));

std::shared_ptr<DPPCmdHeader> elem0 = make_shared<DPPCmdHeader>();
std::shared_ptr<DPPCmdHeader> elem1 = make_shared<DPPCmdHeader>();
// 断言消息解析后处理情况
EXPECT_CALL(*(m_handler.get()), DispatchTask(testing::_))
    .Times(2)
    .WillOnce(SaveArgPointee<0>(elem0))
    .WillOnce(SaveArgPointee<0>(elem1));
```

#### 非虚函数Mock

非虚函数可通过创建具有相同**函数签名**的类mock，然后**模板化代码**，在生产代码中使用原始类型，在测试代码中使用mock类型。参考[这里](https://github.com/google/googletest/blob/master/googlemock/docs/cook_book.md#MockingNonVirtualMethods)

## 参考

- [gMock Cookbook](https://github.com/google/googletest/blob/master/googlemock/docs/cook_book.md)
