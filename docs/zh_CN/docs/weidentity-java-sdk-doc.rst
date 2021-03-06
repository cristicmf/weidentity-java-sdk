.. role:: raw-html-m2r(raw)
   :format: html

.. _weidentity-java-sdk-doc:

WeIdentity JAVA SDK文档
=======================

总体介绍
--------

WeIdentity Java SDK提供了一整套对WeIdentity进行管理操作的Java库。目前，SDK支持本地密钥管理、数字身份标识（WeIdentity DID）管理、电子凭证（WeIdentity Credential）管理、授权机构（Authority Issuer）管理、CPT管理等功能，同时也提供基于FISCO-BCOS的区块链交互、智能合约的部署与调用。未来还将支持更丰富的功能和应用。

术语
----

* 请参阅：`术语表 <https://weidentity.readthedocs.io/zh_CN/latest/docs/terminologies.html>`_

部署SDK
-------

* `WeIdentity JAVA SDK 安装部署文档 <https://weidentity.readthedocs.io/projects/javasdk/zh_CN/latest/docs/weidentity-installation.html>`_   

* 开始使用之前，再次确认启动FISCO-BCOS节点已启动，确保端口可以访问。

整体过程快速上手
----------------

按照以下流程可以完整地体验本SDK的核心功能：


#. 注册DID：通过WeIdService的createWeId()生成一个WeIdentity DID并注册到链上；
#. 设置DID属性：分别调用WeIdService的set方法组，为此DID设置公钥、认证方式、服务端点等属性；
#. 查询DID属性：调用WeIdService的getWeIdDocumentJson()查阅生成的WeIdentity DID数据；
#. 注册授权机构：通过AuthorityIssuerService的registerAuthorityIssuer()把生成的WeIdentity DID注册成一个授权机构；
#. 查询授权机构：调用AuthorityIssuerService的queryAuthorityIssuerInfo()查阅生成的授权机构数据；
#. 注册CPT：通过CptService的registerCpt()，通过之前生成的WeIdentity DID身份创建一个你喜欢的CPT模板；
#. 查询CPT：调用CptService的queryCpt()查阅生成的CPT模板；
#. 生成凭证：通过CredentialService的CreateCredential()，根据CPT模板，生成一份Credential；
#. 查询凭证：调用CredentialService的VerifyCredential()，验证此Credential是否合法；
#. 凭证存证上链：调用EvidenceService的CreateEvidence()，将之前生成的Credential生成一份Hash存证上链；
#. 验证链上凭证存证：调用EvidenceService的VerifyEvidence()，和链上对比，验证Credential是否被篡改。

代码结构说明
------------

.. code-block:: text

   ├─ config：FISCO-BCOS的合约配置
   ├─ constant：系统常量相关
   └─ contract：通过FISCO-BCOS Web3sdk生成的合约Java接口文件
      └─ deploy: 合约部署相关
   └─ protocol：接口参数相关定义
      ├─ base: 基础数据类型定义
      ├─ request: 接口入参定义
      └─ response: 接口出参定义
   ├─ rpc：接口定义
   ├─ service：接口相关实现
   └─ util：工具类实现

接口简介
--------

整体上，WeIdentity Java SDK包括五个主要的接口，它们分别是：AuthorityIssuerService、CptService、CredentialService、WeIdService、EvidenceService。


* AuthorityIssuerService

在WeIdentity的整体架构中，存在着可信的“授权机构”这一角色。一般来说，授权机构特指那些广为人知的、具有一定公信力的、并且有相对频繁签发Credential需求的实体。

本接口提供了对这类授权签发Credential的机构的注册、移除、查询信息等操作。


* CptService

任何凭证的签发，都需要将数据转换成已经注册的CPT (Claim Protocol Type)格式规范，也就是所谓的“标准化格式化数据”。相关机构事先需要注册好CPT，在此之后，签发机构会根据CPT提供符合格式的数据，进而进行凭证的签发。

本接口提供了对CPT的注册、更新、查询等操作。


* CredentialService

凭证签发相关功能的核心接口。

本接口提供凭证的签发和验证操作。


* WeIdService

WeIdentity DID相关功能的核心接口。

本接口提供WeIdentity DID的创建、获取信息、设置属性等相关操作。


* EvidenceService

凭证存证上链的相关接口。

本接口提供凭证的Hash存证的生成上链、链上查询及校验等操作。


接口列表
--------

AuthorityIssuerService
^^^^^^^^^^^^^^^^^^^^^^

1. registerAuthorityIssuer
~~~~~~~~~~~~~~~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.AuthorityIssuerService.registerAuthorityIssuer
   接口定义:ResponseData<Boolean> registerAuthorityIssuer(RegisterAuthorityIssuerArgs args)
   接口描述: 注册成为权威机构。
   注意：这是一个需要权限的操作，目前只有合约的部署者（一般为SDK）才能正确执行。

**接口入参**\ : com.webank.weid.protocol.request.RegisterAuthorityIssuerArgs

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - authorityIssuer
     - AuthorityIssuer
     - Y
     - JavaBean
     - AuthorityIssuer信息，见下
   * - weIdPrivateKey
     - WeIdPrivateKey
     - Y
     - JavaBean
     - 交易私钥，见下


com.webank.weid.protocol.base.AuthorityIssuer

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - weId
     - String
     - Y
     - 授权机构WeIdentity DID
     - 
   * - name
     - String
     - Y
     - 授权机构名称
     - 
   * - created
     - Long
     - N
     - 创建日期
     - 注册时不需要传入，SDK内置默认为当前时间
   * - accValue
     - String
     - Y
     - 授权方累积判定值
     -


com.webank.weid.protocol.base.WeIdPrivateKey

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - privateKey
     - String
     - Y
     - 私钥值
     - 使用十进制数字表示


**接口返回**\ :     com.webank.weid.protocol.response.ResponseData\<Boolean>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - Boolean
     - 返回结果值
     - 


**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - WEID_INVALID
     - 100101
     - 无效的WeIdentity DID
   * - AUTHORITY_ISSUER_ERROR
     - 100200
     - 授权标准异常
   * - AUTHORITY_ISSUER_PRIVATE_KEY_ILLEGAL
     - 100202
     - 私钥格式非法
   * - AUTHORITY_ISSUER_ADDRESS_MISMATCH
     - 100204
     - 地址不匹配
   * - AUTHORITY_ISSUER_OPCODE_MISMATCH
     - 100205
     - 操作码不匹配
   * - AUTHORITY_ISSUER_NAME_ILLEGAL
     - 100206
     - 名称格式非法
   * - AUTHORITY_ISSUER_ACCVALUE_ILLEAGAL
     - 100207
     - 累计值格式非法
   * - TRANSACTION_TIMEOUT
     - 160001
     - 超时
   * - TRANSACTION_EXECUTE_ERROR
     - 160002
     - 交易错误
   * - ILLEGAL_INPUT
     - 160004
     - 参数为空
   * - AUTHORITY_ISSUER_CONTRACT_ERROR_ALREADY_EXIST
     - 500201
     - 授权人已经存在
   * - AUTHORITY_ISSUER_CONTRACT_ERROR_NO_PERMISSION
     - 500203
     - 授权人没有权限


**调用示例**

.. code-block:: java

   AuthorityIssuerService authorityIssuerService = new AuthorityIssuerServiceImpl();

   AuthorityIssuer authorityIssuer = new AuthorityIssuer();
   authorityIssuer.setWeId("did:weid:0x0106595955ce4713fd169bfa68e599eb99ca2e9f");
   authorityIssuer.setName("webank1");
   authorityIssuer.setAccValue("0");

   WeIdPrivateKey weIdPrivateKey = new WeIdPrivateKey();
   weIdPrivateKey.setPrivateKey("84259158061731800175730035500197147557630375762366333000754891654353899157503");

   RegisterAuthorityIssuerArgs registerAuthorityIssuerArgs = new RegisterAuthorityIssuerArgs();
   registerAuthorityIssuerArgs.setAuthorityIssuer(authorityIssuer);
   registerAuthorityIssuerArgs.setWeIdPrivateKey(weIdPrivateKey);

   ResponseData<Boolean> response = authorityIssuerService.registerAuthorityIssuer(registerAuthorityIssuerArgs);


.. code-block:: text

   返回结果如：
   result: true
   errorCode: 0
   errorMessage: success


**时序图**

.. mermaid::

   sequenceDiagram
   participant 调用者
   participant AuthorityIssuerService
   participant WeIdService
   participant 区块链节点
   调用者->>AuthorityIssuerService: 调用RegisterAuthorityIssuer()
   AuthorityIssuerService->>AuthorityIssuerService: 入参非空、格式及合法性检查
   opt 入参校验失败
   AuthorityIssuerService-->>调用者: 报错，提示参数不合法并退出
   end
   AuthorityIssuerService->>WeIdService: 查询WeIdentity DID存在性
   WeIdService->>区块链节点: 链上查询WeIdentity DID属性
   区块链节点-->>WeIdService: 返回查询结果
   WeIdService-->>AuthorityIssuerService: 返回查询结果
   opt 在链上不存在
   AuthorityIssuerService-->>调用者: 报错并退出
   end
   AuthorityIssuerService->>区块链节点: 加载私钥，调用注册合约
   opt 身份校验
   Note over 区块链节点: 如果传入WeIdentity DID在链上不存在
   区块链节点->>区块链节点: 报错并退出
   end
   区块链节点->>区块链节点: 权限检查，执行合约写入AuthorityIssuer信息
   区块链节点-->>AuthorityIssuerService: 返回合约执行结果
   AuthorityIssuerService->>AuthorityIssuerService: 解析合约事件
   opt 失败，地址无效或无权限
   AuthorityIssuerService-->>调用者: 报错并退出
   end
   AuthorityIssuerService-->>调用者: 返回成功


----

2. removeAuthorityIssuer
~~~~~~~~~~~~~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.AuthorityIssuerService.removeAuthorityIssuer
   接口定义:ResponseData<Boolean> removeAuthorityIssuer(RemoveAuthorityIssuerArgs args)
   接口描述: 注销权威机构。
   注意：这是一个需要权限的操作，目前只有合约的部署者（一般为SDK）才能正确执行。

**接口入参**\ :  com.webank.weid.protocol.request.RemoveAuthorityIssuerArgs

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - weId
     - String
     - Y
     - WeIdentity DID
     - 授权机构WeIdentity DID
   * - weIdPrivateKey
     - WeIdPrivateKey
     - Y
     - JavaBean
     - 交易私钥，见下


com.webank.weid.protocol.base.WeIdPrivateKey

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - privateKey
     - String
     - Y
     - 私钥值
     - 使用十进制数字表示


**接口返回**\ :     com.webank.weid.protocol.response.ResponseData\<Boolean>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - Boolean
     - 返回结果值
     - 


**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - WEID_INVALID
     - 100101
     - 无效的WeIdentity DID
   * - AUTHORITY_ISSUER_ERROR
     - 100200
     - 授权标准异常
   * - AUTHORITY_ISSUER_PRIVATE_KEY_ILLEGAL
     - 100202
     - 私钥格式非法
   * - AUTHORITY_ISSUER_ADDRESS_MISMATCH
     - 100204
     - 地址不匹配
   * - AUTHORITY_ISSUER_OPCODE_MISMATCH
     - 100205
     - 操作码不匹配
   * - TRANSACTION_TIMEOUT
     - 160001
     - 超时
   * - TRANSACTION_EXECUTE_ERROR
     - 160002
     - 交易错误
   * - ILLEGAL_INPUT
     - 160004
     - 参数为空
   * - AUTHORITY_ISSUER_CONTRACT_ERROR_NOT_EXISTS
     - 500202
     - 授权人信息不存在
   * - AUTHORITY_ISSUER_CONTRACT_ERROR_NO_PERMISSION
     - 500203
     - 授权人没有权限


**调用示例**

.. code-block:: java

   AuthorityIssuerService authorityIssuerService = new AuthorityIssuerServiceImpl();

   WeIdPrivateKey weIdPrivateKey = new WeIdPrivateKey();
   weIdPrivateKey.setPrivateKey("84259158061731800175730035500197147557630375762366333000754891654353899157503");

   RemoveAuthorityIssuerArgs removeAuthorityIssuerArgs = new RemoveAuthorityIssuerArgs();
   removeAuthorityIssuerArgs.setWeId("did:weid:0x0106595955ce4713fd169bfa68e599eb99ca2e9f");
   removeAuthorityIssuerArgs.setWeIdPrivateKey(weIdPrivateKey);

   ResponseData<Boolean> response = authorityIssuerService.removeAuthorityIssuer(removeAuthorityIssuerArgs);


.. code-block:: text

   返回结果如：
   result: true
   errorCode: 0
   errorMessage: success


**时序图**


.. mermaid::

   sequenceDiagram
   participant 调用者
   participant AuthorityIssuerService
   participant 区块链节点
   调用者->>AuthorityIssuerService: 调用RemoverAuthorityIssuer()
   AuthorityIssuerService->>AuthorityIssuerService: 入参非空、格式及合法性检查
   opt 入参校验失败
   AuthorityIssuerService-->>调用者: 报错，提示参数不合法并退出
   end
   AuthorityIssuerService->>区块链节点: 加载交易私钥，调用移除合约
   区块链节点->>区块链节点: 权限检查，执行合约删除WeIdentity DID信息
   区块链节点-->>AuthorityIssuerService: 返回合约执行结果
   AuthorityIssuerService->>AuthorityIssuerService: 解析合约事件
   opt 失败，地址无效或无权限
   AuthorityIssuerService-->>调用者: 报错并退出
   end
   AuthorityIssuerService-->>调用者: 返回成功



----

3. isAuthorityIssuer
~~~~~~~~~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.AuthorityIssuerService.isAuthorityIssuer
   接口定义:ResponseData<Boolean> isAuthorityIssuer(String weId)
   接口描述: 根据WeIdentity DID判断是否为权威机构。

**接口入参**\ :    String

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - weId
     - String
     - Y
     - WeIdentity DID
     - 用于搜索权限发布者


**接口返回**\ :     com.webank.weid.protocol.response.ResponseData\<Boolean>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - Boolean
     - 返回结果值
     - 


**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - WEID_INVALID
     - 100101
     - 无效的WeIdentity DID
   * - AUTHORITY_ISSUER_ERROR
     - 100200
     - 授权标准异常
   * - TRANSACTION_TIMEOUT
     - 160001
     - 超时
   * - TRANSACTION_EXECUTE_ERROR
     - 160002
     - 交易错误
   * - AUTHORITY_ISSUER_CONTRACT_ERROR_NOT_EXISTS
     - 500202
     - 实体不存在


**调用示例**

.. code-block:: java

   AuthorityIssuerService authorityIssuerService = new AuthorityIssuerServiceImpl();

   String weId = "did:weid:0x0106595955ce4713fd169bfa68e599eb99ca2e9f";
   ResponseData<Boolean> response = authorityIssuerService.isAuthorityIssuer(weId);


.. code-block:: text

   返回结果如：
   result: true
   errorCode: 0
   errorMessage: success


**时序图**

.. mermaid::

   sequenceDiagram
   participant 调用者
   participant AuthorityIssuerService
   participant 区块链节点
   调用者->>AuthorityIssuerService: 调用IsAuthorityIssuer()
   AuthorityIssuerService->>AuthorityIssuerService: 入参非空、格式及合法性检查
   opt 入参校验失败
   AuthorityIssuerService-->>调用者: 报错，提示参数不合法并退出
   end
   AuthorityIssuerService->>区块链节点: 调用查询是否为授权机构合约
   区块链节点->>区块链节点: 执行合约通过WeIdentity DID查询
   区块链节点-->>AuthorityIssuerService: 返回查询结果
   AuthorityIssuerService-->>调用者: 返回是/否


----

4. queryAuthorityIssuerInfo
~~~~~~~~~~~~~~~~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.AuthorityIssuerService.queryAuthorityIssuerInfo
   接口定义:ResponseData<AuthorityIssuer> queryAuthorityIssuerInfo(String weId)
   接口描述: 根据WeIdentity DID查询权威机构信息。

**接口入参**\ :    String

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - weId
     - String
     - Y
     - WeIdentity DID
     - 用于搜索权限发布者


**接口返回**\ :    com.webank.weid.protocol.response.ResponseData\<AuthorityIssuer>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - AuthorityIssuer
     - JavaBean
     - 授权机构信息，见下


com.webank.weid.protocol.base.AuthorityIssuer

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - weId
     - String
     - Y
     - 授权机构WeIdentity DID
     - 
   * - name
     - String
     - Y
     - 授权机构名称
     - 
   * - created
     - Long
     - Y
     - 创建日期
     - 
   * - accValue
     - String
     - Y
     - 授权方累积判定值
     - 


**注意**\ ：因为Solidity 0.4.4的限制，无法正确的返回accValue，因此这里取得的accValue一定为空字符串。未来会进行修改。

**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - WEID_INVALID
     - 100101
     - 无效的WeIdentity DID
   * - AUTHORITY_ISSUER_ERROR
     - 100200
     - 授权标准异常
   * - TRANSACTION_TIMEOUT
     - 160001
     - 超时
   * - TRANSACTION_EXECUTE_ERROR
     - 160002
     - 交易错误
   * - AUTHORITY_ISSUER_CONTRACT_ERROR_NOT_EXISTS
     - 500202
     - 实体不存在


**调用示例**

.. code-block:: java

   AuthorityIssuerService authorityIssuerService = new AuthorityIssuerServiceImpl();

   String weId = "did:weid:0x0106595955ce4713fd169bfa68e599eb99ca2e9f";
   ResponseData<AuthorityIssuer> response = authorityIssuerService.queryAuthorityIssuerInfo(weId);


.. code-block:: text

   返回数据如：
   result:(com.webank.weid.protocol.base.AuthorityIssuer)
      weId: did:weid:0x0106595955ce4713fd169bfa68e599eb99ca2e9f
      name: webank1
      created: 1539239136000
      accValue:
   errorCode: 0
   errorMessage: success

**时序图**

.. mermaid::

   sequenceDiagram
   participant 调用者
   participant AuthorityIssuerService
   participant 区块链节点
   调用者->>AuthorityIssuerService: 调用queryAuthorityIssuerInfo()
   AuthorityIssuerService->>AuthorityIssuerService: 入参非空、格式及合法性检查
   opt 入参校验失败
   AuthorityIssuerService-->>调用者: 报错，提示参数不合法并退出
   end
   AuthorityIssuerService->>区块链节点: 调用查询详细信息合约
   区块链节点->>区块链节点: 执行合约通过WeIdentity DID查询
   区块链节点-->>AuthorityIssuerService: 返回查询结果
   AuthorityIssuerService-->>调用者: 返回查询结果（非授权机构则无）

----

CptService
^^^^^^^^^^

1. registerCpt
~~~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.CptService.registerCpt
   接口定义:ResponseData<CptBaseInfo> registerCpt(CptMapArgs args)
   接口描述: 传入WeIdentity DID，JsonSchema(Map类型) 和其对应的私钥，链上注册CPT，返回CPT编号和版本。

**接口入参**\ :    com.webank.weid.protocol.request.CptMapArgs

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - weIdAuthentication
     - WeIdAuthentication
     - Y
     - 认证信息，包含WeIdentity DID和私钥
     - 用于WeIdentity DID的身份认证
   * - cptJsonSchema
     - Map<String, Object>
     - Y
     - Map类型的JsonSchema信息
     - 基本使用见调用示例


com.webank.weid.protocol.base.WeIdAuthentication

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - weId
     - String
     - Y
     - CPT发布者的WeIdentity DID
     - WeIdentity DID的格式传入
   * - weIdPrivateKey
     - WeIdPrivateKey
     - Y
     - JavaBean
     - 交易私钥，见下


com.webank.weid.protocol.base.WeIdPrivateKey

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - privateKey
     - String
     - Y
     - 私钥值
     - 使用十进制数字表示


**接口返回**\ :    com.webank.weid.protocol.response.ResponseData\<CptBaseInfo>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 此接口返回的code
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - CptBaseInfo
     - JavaBean
     - CPT基础数据，见下


com.webank.weid.protocol.base.CptBaseInfo

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - cptId
     - Integer
     - cpId编号
     - 
   * - cptVersion
     - Integer
     - 版本号
     - 


**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - WEID_INVALID
     - 100101
     - WeIdentity DID无效
   * - WEID_PRIVATEKEY_INVALID
     - 100103
     - 私钥无效
   * - WEID_PRIVATEKEY_DOES_NOT_MATCH
     - 100106
     - 私钥与WeIdentity DID不匹配
   * - WEID_AUTHORITY_INVALID
     - 100109
     - 授权信息无效
   * - CPT_JSON_SCHEMA_INVALID
     - 100301
     - schema无效
   * - CPT_JSON_SCHEMA_NULL
     - 100302
     - schema为null
   * - CPT_EVENT_LOG_NULL
     - 100304
     - 交易日志异常
   * - TRANSACTION_TIMEOUT
     - 160001
     - 超时
   * - TRANSACTION_EXECUTE_ERROR
     - 160002
     - 交易错误
   * - UNKNOW_ERROR
     - 160003
     - 未知异常
   * - ILLEGAL_INPUT
     - 160004
     - 参数为空
   * - CPT_ID_AUTHORITY_ISSUER_EXCEED_MAX
     - 500302
     - 为权威机构生成的cptId超过上限
   * - CPT_PUBLISHER_NOT_EXIST
     - 500303
     - CPT发布者的WeIdentity DID不存在


**调用示例**

.. code-block:: java

   CptService cptService = new CptServiceImpl();

   HashMap<String, Object> cptJsonSchema = new HashMap<String, Object>(3);
   cptJsonSchema.put(JsonSchemaConstant.TITLE_KEY, "cpt template");
   cptJsonSchema.put(JsonSchemaConstant.DESCRIPTION_KEY, "this is a cpt template");

   HashMap<String, Object> propertitesMap1 = new HashMap<String, Object>(2);
   propertitesMap1.put(JsonSchemaConstant.TYPE_KEY, JsonSchemaConstant.DATE_TYPE_STRING);
   propertitesMap1.put(JsonSchemaConstant.DESCRIPTION_KEY, "this is name");

   String[] genderEnum = {"F", "M"};
   HashMap<String, Object> propertitesMap2 = new HashMap<String, Object>(2);
   propertitesMap2.put(JsonSchemaConstant.TYPE_KEY, JsonSchemaConstant.DATE_TYPE_STRING);
   propertitesMap2.put(JsonSchemaConstant.DATE_TYPE_ENUM, genderEnum);

   HashMap<String, Object> propertitesMap3 = new HashMap<String, Object>(2);
   propertitesMap3.put(JsonSchemaConstant.TYPE_KEY, JsonSchemaConstant.DATE_TYPE_NUMBER);
   propertitesMap3.put(JsonSchemaConstant.DESCRIPTION_KEY, "this is age");

   HashMap<String, Object> cptJsonSchemaKeys = new HashMap<String, Object>(3);
   cptJsonSchemaKeys.put("name", propertitesMap1);
   cptJsonSchemaKeys.put("gender", propertitesMap2);
   cptJsonSchemaKeys.put("age", propertitesMap3);
   cptJsonSchema.put(JsonSchemaConstant.PROPERTIES_KEY, cptJsonSchemaKeys);

   String[] genderRequired = {"name", "gender"};
   cptJsonSchema.put(JsonSchemaConstant.REQUIRED_KEY, genderRequired);

   WeIdPrivateKey weIdPrivateKey = new WeIdPrivateKey();
   weIdPrivateKey.setPrivateKey("84259158061731800175730035500197147557630375762366333000754891654353899157503");

   WeIdAuthentication weIdAuthentication = new WeIdAuthentication();
   weIdAuthentication.setWeId("did:weid:0x0106595955ce4713fd169bfa68e599eb99ca2e9f");
   weIdAuthentication.setWeIdPrivateKey(weIdPrivateKey);

   CptMapArgs cptMapArgs = new CptMapArgs();
   cptMapArgs.setCptJsonSchema(cptJsonSchema);
   cptMapArgs.setWeIdAuthentication(weIdAuthentication);

   ResponseData<CptBaseInfo> response = cptService.registerCpt(cptMapArgs);


.. code-block:: text

   返回数据如下：
   result:(com.webank.weid.protocol.response.CptBaseInfo)
      cptId: 148
      cptVersion: 1
   errorCode: 0
   errorMessage: success

**时序图**

（同时也包含重载updateCpt时序）

.. mermaid::

   sequenceDiagram
   调用者->>WeIdentity SDK : 传入自己已有的WeIdentity DID及对应的私钥，及其jsonSchema，调用registerCpt来注册CPT。
   opt 参数校验
   Note over WeIdentity SDK:如果WeIdentity DID或者私钥为空或不匹配
   WeIdentity SDK->>WeIdentity SDK:报错，提示参数不合法并退出
   end
   WeIdentity SDK->>区块链节点: 将java对象转换为合约所需的字段，调用智能合约，将CPT信息上链
   opt 身份校验
   Note over 区块链节点:如果传入WeIdentity DID在链上不存在
   区块链节点->>区块链节点:报错，提示WeIdentity DID不存在并退出
   end
   区块链节点->>区块链节点:写入CPT信息
   区块链节点-->>WeIdentity SDK:返回
   WeIdentity SDK-->>调用者:返回调用结果

----

2. registerCpt
~~~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.CptService.registerCpt
   接口定义:ResponseData<CptBaseInfo> registerCpt(CptStringArgs args)
   接口描述: 传入WeIdentity DID，JsonSchema(String类型) 和其对应的私钥，链上注册CPT，返回CPT编号和版本。

**接口入参**\ :    com.webank.weid.protocol.request.CptStringArgs

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - weIdAuthentication
     - WeIdAuthentication
     - Y
     - 认证信息，包含WeIdentity DID和私钥
     - 用于WeIdentity DID的身份认证
   * - cptJsonSchema
     - String
     - Y
     - 字符串类型的JsonSchema信息
     - 基本使用见调用示例


com.webank.weid.protocol.base.WeIdAuthentication

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - weId
     - String
     - Y
     - CPT发布者的WeIdentity DID
     - WeIdentity DID的格式传入
   * - weIdPrivateKey
     - WeIdPrivateKey
     - Y
     - JavaBean
     - 交易私钥，见下


com.webank.weid.protocol.base.WeIdPrivateKey

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - privateKey
     - String
     - Y
     - 私钥值
     - 使用十进制数字表示


**接口返回**\ :    com.webank.weid.protocol.response.ResponseData\<CptBaseInfo>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 此接口返回的code
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - CptBaseInfo
     - JavaBean
     - CPT基础数据，见下


com.webank.weid.protocol.base.CptBaseInfo

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - cptId
     - Integer
     - cpId编号
     - 
   * - cptVersion
     - Integer
     - 版本号
     - 


.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - WEID_INVALID
     - 100101
     - WeIdentity DID无效
   * - WEID_PRIVATEKEY_INVALID
     - 100103
     - 私钥无效
   * - WEID_PRIVATEKEY_DOES_NOT_MATCH
     - 100106
     - 私钥与WeIdentity DID不匹配
   * - WEID_AUTHORITY_INVALID
     - 100109
     - 授权信息无效
   * - CPT_JSON_SCHEMA_INVALID
     - 100301
     - schema无效
   * - CPT_JSON_SCHEMA_NULL
     - 100302
     - schema为null
   * - CPT_EVENT_LOG_NULL
     - 100304
     - 交易日志异常
   * - TRANSACTION_TIMEOUT
     - 160001
     - 超时
   * - TRANSACTION_EXECUTE_ERROR
     - 160002
     - 交易错误
   * - UNKNOW_ERROR
     - 160003
     - 未知异常
   * - ILLEGAL_INPUT
     - 160004
     - 参数为空
   * - CPT_ID_AUTHORITY_ISSUER_EXCEED_MAX
     - 500302
     - 为权威机构生成的cptId超过上限
   * - CPT_PUBLISHER_NOT_EXIST
     - 500303
     - CPT发布者的WeIdentity DID不存在


**调用示例**

.. code-block:: java

   CptService cptService = new CptServiceImpl();

   String jsonSchema = "{\"properties\" : {\"name\": {\"type\": \"string\",\"description\": \"the name of certificate owner\"},\"gender\": {\"enum\": [\"F\", \"M\"],\"type\": \"string\",\"description\": \"the gender of certificate owner\"}, \"age\": {\"type\": \"number\", \"description\": \"the age of certificate owner\"}},\"required\": [\"name\", \"age\"]}";

   WeIdPrivateKey weIdPrivateKey = new WeIdPrivateKey();
   weIdPrivateKey.setPrivateKey("84259158061731800175730035500197147557630375762366333000754891654353899157503");

   WeIdAuthentication weIdAuthentication = new WeIdAuthentication();
   weIdAuthentication.setWeId("did:weid:0x0106595955ce4713fd169bfa68e599eb99ca2e9f");
   weIdAuthentication.setWeIdPrivateKey(weIdPrivateKey);

   CptStringArgs cptStringArgs = new CptStringArgs();
   cptStringArgs.setCptJsonSchema(jsonSchema);
   cptStringArgs.setWeIdAuthentication(weIdAuthentication);

   ResponseData<CptBaseInfo> response = cptService.registerCpt(cptStringArgs);


.. code-block:: text

   返回数据如下：
   result:(com.webank.weid.protocol.response.CptBaseInfo)
      cptId: 149
      cptVersion: 1
   errorCode: 0
   errorMessage: success

----

3. queryCpt
~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.CptService.queryCpt
   接口定义:ResponseData<Cpt> queryCpt(Integer cptId)
   接口描述: 根据CPT编号查询CPT信息。

**接口入参**\ :    java.lang.Integer

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - cptId
     - Integer
     - Y
     - cptId编号
     -


**接口返回**\ :    com.webank.weid.protocol.response.ResponseData\<Cpt>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 此接口返回的code
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - Cpt
     - JavaBean
     - CPT内容，见下


com.webank.weid.protocol.base.Cpt

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - cptJsonSchema
     - Map<String, Object>
     - Map类型的cptJsonSchema信息
     - 
   * - cptBaseInfo
     - CptBaseInfo
     - JavaBean
     - CPT基础数据，见下
   * - cptMetaData
     - CptMetaData
     - JavaBean
     - CPT元数据内部类，见下


com.webank.weid.protocol.base.CptBaseInfo

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - cptId
     - Integer
     - cpId编号
     - 
   * - cptVersion
     - Integer
     - 版本号
     - 


com.webank.weid.protocol.base.Cpt.MetaData

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - cptPublisher
     - String
     - CPT发布者的WeIdentity DID
     - WeIdentity DID格式数据
   * - cptSignature
     - String
     - 签名数据
     - cptPublisher与cptJsonSchema拼接的签名数据
   * - updated
     - long
     - 更新时间
     - 
   * - created
     - long
     - 创建日期
     - 


**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - TRANSACTION_EXECUTE_ERROR
     - 160002
     - 交易错误
   * - UNKNOW_ERROR
     - 160003
     - 未知异常
   * - ILLEGAL_INPUT
     - 160004
     - 参数非法
   * - CPT_NOT_EXISTS
     - 500301
     - CPT不存在


**调用示例**

.. code-block:: java

   CptService cptService = new CptServiceImpl();

   Integer cptId = Integer.valueOf(147);
   ResponseData<Cpt> response = cptService.queryCpt(cptId);


.. code-block:: text

   返回数据如下：
   result:(com.webank.weid.protocol.base.Cpt)
      cptBaseInfo:(com.webank.weid.protocol.base.CptBaseInfo)
         cptId: 147
         cptVersion: 1
      cptJsonSchema:(java.util.HashMap)
         $schema: http://json-schema.org/draft-04/schema#
         type: object
         properties:(java.util.LinkedHashMap)
            age:(java.util.LinkedHashMap)
               description: this is age
               type: number
            gender:(java.util.LinkedHashMap)
               enum:(java.util.ArrayList)
                  [0]:F
                  [1]:M
               type: string
            name:(java.util.LinkedHashMap)
               description: this is name
               type: string
         required:(java.util.ArrayList)
            [0]:name
            [1]:gender
      metaData:(com.webank.weid.protocol.base.Cpt$MetaData)
         cptPublisher: did:weid:0xcf4ba44b03549f0eaf1a422783b45b22aa0cc25a
         cptSignature: HJz29zBofHrJI9m6ZrXdBXPxLTj4ynW50DoDekI93buPb1RWu47w4imtShCBF/+wxCsHVg0Pw10NzoLgvVhSn0Y=
         created: 1551344040521
         updated: 0
   errorCode: 0
   errorMessage: success


**时序图**


.. mermaid::

   sequenceDiagram
   调用者->>WeIdentity SDK : 传入指定的cptId
   opt 参数校验
   Note over WeIdentity SDK:检查传入的cptId是否为空或负数
   WeIdentity SDK->>WeIdentity SDK:报错，提示weid不合法并退出
   end
   WeIdentity SDK->>区块链节点: 调用合约查询链上的指定cpt对应的信息
   区块链节点-->>WeIdentity SDK:返回
   WeIdentity SDK->>WeIdentity SDK:根据合约返回的值构建返回的java对象
   WeIdentity SDK-->>调用者:返回调用结果


----

4. updateCpt
~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.CptService.updateCpt
   接口定义:ResponseData<CptBaseInfo> updateCpt(CptMapArgs args, Integer cptId)
   接口描述: 传入cptId，JsonSchema(Map类型)，WeIdentity DID，WeIdentity DID所属私钥，进行更新CPT信息，更新成功版本自动+1。

**接口入参**\ :    com.webank.weid.protocol.request.CptMapArgs，Integer

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - args
     - CptMapArgs
     - Y
     - CPT信息
     - 具体见下
   * - cptId
     - Integer
     - Y
     - 发布的CPT编号
     - 

com.webank.weid.protocol.request.CptMapArgs

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - weIdAuthentication
     - WeIdAuthentication
     - Y
     - 认证信息，包含WeIdentity DID和私钥
     - 用于WeIdentity DID的身份认证
   * - cptJsonSchema
     - Map<String, Object>
     - Y
     - Map类型的JsonSchema信息
     - 基本使用见调用示例


com.webank.weid.protocol.base.WeIdAuthentication

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - weId
     - String
     - Y
     - CPT发布者的WeIdentity DID
     - WeIdentity DID的格式传入
   * - weIdPrivateKey
     - WeIdPrivateKey
     - Y
     - JavaBean
     - 交易私钥，见下


com.webank.weid.protocol.base.WeIdPrivateKey

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - privateKey
     - String
     - Y
     - 私钥值
     - 使用十进制数字表示


**接口返回**\ :    com.webank.weid.protocol.response.ResponseData\<CptBaseInfo>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 此接口返回的code
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - CptBaseInfo
     - JavaBean
     - CPT基础数据，见下


com.webank.weid.protocol.base.CptBaseInfo

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - cptId
     - Integer
     - cpId编号
     - 
   * - cptVersion
     - Integer
     - 版本号
     -  


**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - WEID_INVALID
     - 100101
     - WeIdentity DID无效
   * - WEID_PRIVATEKEY_INVALID
     - 100103
     - 私钥无效
   * - WEID_PRIVATEKEY_DOES_NOT_MATCH
     - 100106
     - 私钥与WeIdentity DID不匹配
   * - WEID_AUTHORITY_INVALID
     - 100109
     - 授权信息无效
   * - CPT_JSON_SCHEMA_INVALID
     - 100301
     - schema无效
   * - CPT_JSON_SCHEMA_NULL
     - 100302
     - schema为null
   * - CPT_ID_NULL
     - 100303
     - CPT编号为null
   * - CPT_EVENT_LOG_NULL
     - 100304
     - 交易日志异常
   * - TRANSACTION_TIMEOUT
     - 160001
     - 超时
   * - TRANSACTION_EXECUTE_ERROR
     - 160002
     - 交易错误
   * - UNKNOW_ERROR
     - 160003
     - 未知异常
   * - ILLEGAL_INPUT
     - 160004
     - 参数为空
   * - CPT_NOT_EXISTS
     - 500301
     - CPT不存在
   * - CPT_PUBLISHER_NOT_EXIST
     - 500303
     - CPT发布者的WeIdentity DID不存在


**调用示例**

.. code-block:: java

   CptService cptService = new CptServiceImpl();

   HashMap<String, Object> cptJsonSchema = new HashMap<String, Object>(3);
   cptJsonSchema.put(JsonSchemaConstant.TITLE_KEY, "cpt template");
   cptJsonSchema.put(JsonSchemaConstant.DESCRIPTION_KEY, "this is a cpt template");

   HashMap<String, Object> propertitesMap1 = new HashMap<String, Object>(2);
   propertitesMap1.put(JsonSchemaConstant.TYPE_KEY, JsonSchemaConstant.DATE_TYPE_STRING);
   propertitesMap1.put(JsonSchemaConstant.DESCRIPTION_KEY, "this is name");

   String[] genderEnum = {"F", "M"};
   HashMap<String, Object> propertitesMap2 = new HashMap<String, Object>(2);
   propertitesMap2.put(JsonSchemaConstant.TYPE_KEY, JsonSchemaConstant.DATE_TYPE_STRING);
   propertitesMap2.put(JsonSchemaConstant.DATE_TYPE_ENUM, genderEnum);

   HashMap<String, Object> propertitesMap3 = new HashMap<String, Object>(2);
   propertitesMap3.put(JsonSchemaConstant.TYPE_KEY, JsonSchemaConstant.DATE_TYPE_NUMBER);
   propertitesMap3.put(JsonSchemaConstant.DESCRIPTION_KEY, "this is age");

   HashMap<String, Object> cptJsonSchemaKeys = new HashMap<String, Object>(3);
   cptJsonSchemaKeys.put("name", propertitesMap1);
   cptJsonSchemaKeys.put("gender", propertitesMap2);
   cptJsonSchemaKeys.put("age", propertitesMap3);
   cptJsonSchema.put(JsonSchemaConstant.PROPERTIES_KEY, cptJsonSchemaKeys);

   String[] genderRequired = {"name", "gender"};
   cptJsonSchema.put(JsonSchemaConstant.REQUIRED_KEY, genderRequired);

   WeIdPrivateKey weIdPrivateKey = new WeIdPrivateKey();
   weIdPrivateKey.setPrivateKey("84259158061731800175730035500197147557630375762366333000754891654353899157503");

   WeIdAuthentication weIdAuthentication = new WeIdAuthentication();
   weIdAuthentication.setWeId("did:weid:0x0106595955ce4713fd169bfa68e599eb99ca2e9f");
   weIdAuthentication.setWeIdPrivateKey(weIdPrivateKey);

   CptMapArgs cptMapArgs = new CptMapArgs();
   cptMapArgs.setCptJsonSchema(cptJsonSchema);
   cptMapArgs.setWeIdAuthentication(weIdAuthentication);

   Integer cptId = Integer.valueOf(148);

   ResponseData<CptBaseInfo> response = cptService.updateCpt(cptMapArgs, cptId);


.. code-block:: text

   返回数据如下：
   result:(com.webank.weid.protocol.response.CptBaseInfo)
      cptId: 148
      cptVersion: 3
   errorCode: 0
   errorMessage: success

**时序图**

（同时也包含重载updateCpt时序）

.. mermaid::

   sequenceDiagram
   调用者->>WeIdentity SDK : 传入自己已有的WeIdentity DID及对应的私钥，及其需新版本的jsonSchema，调用updateCpt来更新CPT。
   opt 参数校验
   Note over WeIdentity SDK:如果WeIdentity DID或者私钥为空或不匹配
   WeIdentity SDK->>WeIdentity SDK:报错，提示参数不合法并退出
   end
   WeIdentity SDK->>区块链节点: 将java对象转换为合约所需的字段，调用智能合约，将更新的CPT信息上链
   opt 身份校验
   Note over 区块链节点:如果传入WeIdentity DID在链上不存在
   区块链节点->>区块链节点:报错，提示WeIdentity DID不存在并退出
   end
   区块链节点->>区块链节点:写入CPT更新信息
   区块链节点-->>WeIdentity SDK:返回
   WeIdentity SDK-->>调用者:返回调用结果

----

5. updateCpt
~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.CptService.updateCpt
   接口定义:ResponseData<CptBaseInfo> updateCpt(CptStringArgs args, Integer cptId)
   接口描述: 传入cptId，JsonSchema(String类型)，WeIdentity DID，WeIdentity DID所属私钥，进行更新CPT信息，更新成功版本自动+1。

**接口入参**\ :    com.webank.weid.protocol.request.CptStringArgs，Integer

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - args
     - CptStringArgs
     - Y
     - CPT信息
     - 具体见下
   * - cptId
     - Integer
     - Y
     - 发布的CPT编号
     - 

com.webank.weid.protocol.request.CptStringArgs

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - weIdAuthentication
     - WeIdAuthentication
     - Y
     - 认证信息，包含WeIdentity DID和私钥
     - 用于WeIdentity DID的身份认证
   * - cptJsonSchema
     - String
     - Y
     - 字符串类型的JsonSchema信息
     - 基本使用见调用示例


com.webank.weid.protocol.base.WeIdAuthentication

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - weId
     - String
     - Y
     - CPT发布者的WeIdentity DID
     - WeIdentity DID的格式传入
   * - weIdPrivateKey
     - WeIdPrivateKey
     - Y
     - JavaBean
     - 交易私钥，见下


com.webank.weid.protocol.base.WeIdPrivateKey

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - privateKey
     - String
     - Y
     - 私钥值
     - 使用十进制数字表示


**接口返回**\ :    com.webank.weid.protocol.response.ResponseData\<CptBaseInfo>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 此接口返回的code
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - CptBaseInfo
     - JavaBean
     - CPT基础数据，见下


com.webank.weid.protocol.base.CptBaseInfo

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - cptId
     - Integer
     - cpId编号
     - 
   * - cptVersion
     - Integer
     - 版本号
     -  


**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - WEID_INVALID
     - 100101
     - WeIdentity DID无效
   * - WEID_PRIVATEKEY_INVALID
     - 100103
     - 私钥无效
   * - WEID_PRIVATEKEY_DOES_NOT_MATCH
     - 100106
     - 私钥与WeIdentity DID不匹配
   * - WEID_AUTHORITY_INVALID
     - 100109
     - 授权信息无效
   * - CPT_JSON_SCHEMA_INVALID
     - 100301
     - schema无效
   * - CPT_JSON_SCHEMA_NULL
     - 100302
     - schema为null
   * - CPT_ID_NULL
     - 100303
     - CPT编号为null
   * - CPT_EVENT_LOG_NULL
     - 100304
     - 交易日志异常
   * - TRANSACTION_TIMEOUT
     - 160001
     - 超时
   * - TRANSACTION_EXECUTE_ERROR
     - 160002
     - 交易错误
   * - UNKNOW_ERROR
     - 160003
     - 未知异常
   * - ILLEGAL_INPUT
     - 160004
     - 参数为空
   * - CPT_NOT_EXISTS
     - 500301
     - CPT不存在
   * - CPT_PUBLISHER_NOT_EXIST
     - 500303
     - CPT发布者的WeIdentity DID不存在


**调用示例**

.. code-block:: java

   CptService cptService = new CptServiceImpl();

   String jsonSchema = "{\"properties\" : {\"name\": {\"type\": \"string\",\"description\": \"the name of certificate owner\"},\"gender\": {\"enum\": [\"F\", \"M\"],\"type\": \"string\",\"description\": \"the gender of certificate owner\"}, \"age\": {\"type\": \"number\", \"description\": \"the age of certificate owner\"}},\"required\": [\"name\", \"age\"]}";

   WeIdPrivateKey weIdPrivateKey = new WeIdPrivateKey();
   weIdPrivateKey.setPrivateKey("84259158061731800175730035500197147557630375762366333000754891654353899157503");

   WeIdAuthentication weIdAuthentication = new WeIdAuthentication();
   weIdAuthentication.setWeId("did:weid:0x0106595955ce4713fd169bfa68e599eb99ca2e9f");
   weIdAuthentication.setWeIdPrivateKey(weIdPrivateKey);

   CptStringArgs cptStringArgs = new CptStringArgs();
   cptStringArgs.setCptJsonSchema(jsonSchema);
   cptStringArgs.setWeIdAuthentication(weIdAuthentication);

   Integer cptId = Integer.valueOf(148);

   ResponseData<CptBaseInfo> response = cptService.updateCpt(cptStringArgs, cptId);


.. code-block:: text

   返回数据如下：
   result:(com.webank.weid.protocol.response.CptBaseInfo)
      cptId: 148
      cptVersion: 4
   errorCode: 0
   errorMessage: success

----

CredentialService
^^^^^^^^^^^^^^^^^

1. createCredential
~~~~~~~~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.CredentialService.createCredential
   接口定义:ResponseData<CredentialWrapper> createCredential(CreateCredentialArgs args)
   接口描述: 创建电子凭证。

**接口入参**\ :   com.webank.weid.protocol.request.CreateCredentialArgs

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - cptId
     - Integer
     - Y
     - CPT编号
     - 
   * - issuer
     - String
     - Y
     - 发行方WeIdentity DID
     - WeIdentity DID格式数据
   * - expirationDate
     - Long
     - Y
     - 到期日
     - 
   * - claim
     - Map<String, Object>
     - Y
     - Map类型的claim数据
     - 凭证所需数据
   * - weIdPrivateKey
     - WeIdPrivateKey
     - Y
     - JavaBean
     - 签名所用Issuer WeIdentity DID私钥，见下


com.webank.weid.protocol.base.WeIdPrivateKey

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - privateKey
     - String
     - Y
     - 私钥值
     - 使用十进制数字表示


**接口返回**\ :    com.webank.weid.protocol.response.ResponseData\<CredentialWrapper>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - CredentialWrapper
     - JavaBean
     - 见下

com.webank.weid.protocol.base.CredentialWrapper

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - credential
     - Credential
     - Y
     - 凭证信息
     - 具体见下
   * - disclosure
     - Map<String, Object>
     - Y
     - 披露属性
     - 默认为全披露

com.webank.weid.protocol.base.Credential

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - context
     - String
     - Y
     - 版本
     - 默认为v1
   * - id
     - String
     - Y
     - 证书编号
     - 
   * - cptId
     - Integer
     - Y
     - cptId
     - 
   * - issuer
     - String
     - Y
     - WeIdentity DID
     - 
   * - issuranceDate
     - Long
     - Y
     - 创建日期
     - 
   * - expirationDate
     - Long
     - Y
     - 到期日期
     - 
   * - signature
     - String
     - Y
     - 签名数据
     - 
   * - claim
     - Map<String, Object>
     - Y
     - Claim数据
     - 


**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - CPT_JSON_SCHEMA_INVALID
     - 100301
     - JsonSchema无效
   * - CREDENTIAL_ERROR
     - 100400
     - Credential标准错误
   * - CREDENTIAL_EXPIRE_DATE_ILLEGAL
     - 100409
     - 到期日期无效
   * - CREDENTIAL_CLAIM_NOT_EXISTS
     - 100410
     - Claim数据不能为空
   * - CREDENTIAL_CLAIM_DATA_ILLEGAL
     - 100411
     - Claim数据无效
   * - CREDENTIAL_PRIVATE_KEY_NOT_EXISTS
     - 100415
     - 私钥为空
   * - CREDENTIAL_CPT_NOT_EXISTS
     - 100416
     - cpt不存在
   * - CREDENTIAL_ISSUER_INVALID
     - 100418
     - WeIdentity DID无效
   * - ILLEGAL_INPUT
     - 160004
     - 参数为空


**调用示例**

.. code-block:: java

   CredentialService credentialService = new CredentialServiceImpl();

   HashMap<String, Object> cptJsonSchemaData = new HashMap<String, Object>(3);
   cptJsonSchemaData.put("name", "zhang san");
   cptJsonSchemaData.put("gender", "F");
   cptJsonSchemaData.put("age", 18);

   CreateCredentialArgs createCredentialArgs = new CreateCredentialArgs();
   createCredentialArgs.setClaim(cptJsonSchemaData);
   createCredentialArgs.setCptId(155);
   createCredentialArgs.setExpirationDate(1551448312461L);
   createCredentialArgs.setIssuer("did:weid:0x30404b47c6c5811d49e28ea2306c804d16618017");

   WeIdPrivateKey weIdPrivateKey = new WeIdPrivateKey();
   weIdPrivateKey.setPrivateKey("84259158061731800175730035500197147557630375762366333000754891654353899157503");

   createCredentialArgs.setWeIdPrivateKey(weIdPrivateKey);

   ResponseData<CredentialWrapper> response = credentialService.createCredential(createCredentialArgs);


.. code-block:: text

   返回结果如：
   result:(com.webank.weid.protocol.base.CredentialWrapper)
      credential:(com.webank.weid.protocol.base.Credential)
         context: v1
         id: d0859e54-0c0d-4320-9bb6-5174b86dd598
         cptId: 155
         issuer: did:weid:0x30404b47c6c5811d49e28ea2306c804d16618017
         issuranceDate: 1551348312461
         expirationDate: 1551434712408
         claim:(java.util.HashMap)
            name: zhang san
            gender: F
            age: 18
         signature: HGBhyAwy+q5C7MjXhRssZNiVOXmGwzNIypzrleEwEZKgeOV1bpmm3v/wl0z6ACdIUGHkIJiCDlLS9NhDwVf/1nY=
      disclosure:(java.util.HashMap)
         name: 1
         gender: 1
         age: 1
   errorCode: 0
   errorMessage: success

**时序图**

.. mermaid::

   sequenceDiagram
   participant 调用者
   participant CredentialService
   调用者->>CredentialService: 调用CreateCredential()
   CredentialService->>CredentialService: 入参非空、格式及合法性检查
   opt 入参校验失败
   CredentialService-->>调用者: 报错，提示参数不合法并退出
   end
   CredentialService->>CredentialService: 生成签发日期、生成数字签名
   CredentialService-->>调用者: 返回凭证

----

2. verify
~~~~~~~~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.CredentialService.verify
   接口定义:ResponseData<Boolean> verify(Credential credential);
   接口描述: 验证凭证是否正确。

**接口入参**\ :   com.webank.weid.protocol.base.Credential

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - context
     - String
     - Y
     - 版本
     - 默认为v1
   * - id
     - String
     - Y
     - 证书编号
     - 
   * - cptId
     - Integer
     - Y
     - cptId
     - 
   * - issuer
     - String
     - Y
     - WeIdentity DID
     - 
   * - issuranceDate
     - Long
     - Y
     - 创建日期
     - 
   * - expirationDate
     - Long
     - Y
     - 到期日期
     - 
   * - signature
     - String
     - Y
     - 签名数据
     - 
   * - claim
     - Map<String, Object>
     - Y
     - Claim数据
     - 


**接口返回**\ :   com.webank.weid.protocol.response.ResponseData\<Boolean>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - Boolean
     - 返回结果值
     - 


**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - CREDENTIAL_ERROR
     - 100400
     - Credential标准错误
   * - CREDENTIAL_NOT_EXISTS
     - 100401
     - Credential入参为空
   * - CREDENTIAL_EXPIRED
     - 100402
     - 过期
   * - CREDENTIAL_ISSUER_MISMATCH
     - 100403
     - issuer与签名不匹配
   * - CREDENTIAL_SIGNATURE_BROKEN
     - 100405
     - 签名破坏
   * - CREDENTIAL_ISSUER_NOT_EXISTS
     - 100407
     - WeIdentity DID不能为空
   * - CREDENTIAL_CREATE_DATE_ILLEGAL
     - 100408
     - 创建日期格式非法
   * - CREDENTIAL_EXPIRE_DATE_ILLEGAL
     - 100409
     - 到期日期格式非法
   * - CREDENTIAL_CLAIM_NOT_EXISTS
     - 100410
     - Claim数据不能为空
   * - CREDENTIAL_CLAIM_DATA_ILLEGAL
     - 100411
     - Claim数据无效
   * - CREDENTIAL_ID_NOT_EXISTS
     - 100412
     - ID为空
   * - CREDENTIAL_CONTEXT_NOT_EXISTS
     - 100413
     - context为空
   * - CREDENTIAL_CPT_NOT_EXISTS
     - 100416
     - cpt不存在
   * - CREDENTIAL_WEID_DOCUMENT_ILLEGAL
     - 100417
     - WeIdentity Document为空
   * - CREDENTIAL_ISSUER_INVALID
     - 100418
     - WeIdentity DID无效
   * - CREDENTIAL_EXCEPTION_VERIFYSIGNATURE
     - 100419
     - 验证签名异常
   * - ILLEGAL_INPUT
     - 160004
     - 参数为空


**调用示例**

.. code-block:: java

   CredentialService credentialService = new CredentialServiceImpl();

   HashMap<String, Object> cptJsonSchemaData = new HashMap<String, Object>(3);
   cptJsonSchemaData.put("name", "zhang san");
   cptJsonSchemaData.put("gender", "F");
   cptJsonSchemaData.put("age", 18);

   Credential credential = new Credential();
   credential.setClaim(cptJsonSchemaData);
   credential.setContext("v1");
   credential.setCptId(155);
   credential.setIssuranceDate(1551348312461L);
   credential.setId("54bc3832-fce7-433a-80c7-ba284635c67a");// 系统生成
   credential.setSignature("HGBhyAwy+q5C7MjXhRssZNiVOXmGwzNIypzrleEwEZKgeOV1bpmm3v/wl0z6ACdIUGHkIJiCDlLS9NhDwVf/1nY=");
   credential.setExpirationDate(1551434712408L);
   credential.setIssuer("did:weid:0x30404b47c6c5811d49e28ea2306c804d16618017");

   ResponseData<Boolean> response = credentialService.verify(credential);


.. code-block:: text

   返回结果如：
   result: false
   errorCode: 0
   errorMessage: success


**时序图**

（同时也包含verifyCredentialWithSpecifiedPubKey时序）

.. mermaid::

   sequenceDiagram
   participant 调用者
   participant CredentialService
   participant CptService
   participant WeIdService
   participant 区块链节点
   调用者->>CredentialService: 调用verify()或verifyCredentialWithSpecifiedPubKey()
   CredentialService->>CredentialService: 入参非空、格式及合法性检查
   opt 入参校验失败
   CredentialService-->>调用者: 报错，提示参数不合法并退出
   end
   CredentialService->>WeIdService: 查询WeIdentity DID存在性
   WeIdService->>区块链节点: 调用智能合约，查询WeIdentity DID属性
   区块链节点-->>WeIdService: 返回查询结果
   WeIdService-->>CredentialService: 返回查询结果
   opt 查询不存在
   CredentialService-->>调用者: 报错并退出
   end
   CredentialService->>CptService: 查询CPT存在性及Claim关联语义
   CptService->>区块链节点: 调用智能合约，查询CPT
   区块链节点-->>CptService: 返回查询结果
   CptService-->>CredentialService: 返回查询结果
   opt 不符合CPT格式要求
   CredentialService-->>调用者: 报错并退出
   end
   CredentialService->>CredentialService: 验证过期、撤销与否
   opt 任一验证失败
   CredentialService-->>调用者: 报错并退出
   end
   opt 未提供验签公钥
   CredentialService->>WeIdService: 查询Issuer对应公钥
   WeIdService->>区块链节点: 调用智能合约，查询Issuer的WeIdentity DID Document
   区块链节点-->>WeIdService: 返回查询结果
   WeIdService-->>CredentialService: 返回查询结果
   end
   CredentialService->>CredentialService: 通过公钥与签名对比，验证Issuer是否签发此凭证
   opt 验证签名失败
   CredentialService-->>调用者: 报错并退出
   end
   CredentialService-->>调用者: 返回成功

----

3. verifyCredentialWithSpecifiedPubKey
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.CredentialService.verifyCredentialWithSpecifiedPubKey
   接口定义: ResponseData<Boolean> verifyCredentialWithSpecifiedPubKey(CredentialWrapper credentialWrapper, WeIdPublicKey weIdPublicKey)
   接口描述: 验证凭证是否正确，需传入公钥。

**接口入参**\ :   

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - credentialWrapper
     - CredentialWrapper
     - Y
     - JavaBean
     - 凭证信息，见下
   * - weIdPublicKey
     - WeIdPublicKey
     - Y
     - JavaBean
     - 公钥信息，见下

com.webank.weid.protocol.base.CredentialWrapper

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - credential
     - Credential
     - Y
     - 凭证信息
     - 具体见下
   * - disclosure
     - Map<String, Object>
     - N
     - 披露属性
     - 默认为全披露


com.webank.weid.protocol.base.Credential

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - context
     - String
     - Y
     - 版本
     - 默认为v1
   * - id
     - String
     - Y
     - 证书编号
     - 
   * - cptId
     - Integer
     - Y
     - cptId
     - 
   * - issuer
     - String
     - Y
     - WeIdentity DID
     - 
   * - issuranceDate
     - Long
     - Y
     - 创建日期
     - 
   * - expirationDate
     - Long
     - Y
     - 到期日期
     - 
   * - signature
     - String
     - Y
     - 签名数据
     - 
   * - claim
     - Map<String, Object>
     - Y
     - Claim数据
     - 
     
     
com.webank.weid.protocol.base.WeIdPublicKey

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - publicKey
     - String
     - 数字公钥
     - 使用十进制数字表示


**接口返回**\ :   com.webank.weid.protocol.response.ResponseData\<Boolean>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - Boolean
     - 返回结果值
     - 


**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - CREDENTIAL_ERROR
     - 100400
     - Credential标准错误
   * - CREDENTIAL_NOT_EXISTS
     - 100401
     - Credential入参为空
   * - CREDENTIAL_EXPIRED
     - 100402
     - 过期
   * - CREDENTIAL_ISSUER_MISMATCH
     - 100403
     - issuer与签名不匹配
   * - CREDENTIAL_SIGNATURE_BROKEN
     - 100405
     - 签名破坏
   * - CREDENTIAL_ISSUER_NOT_EXISTS
     - 100407
     - WeIdentity DID不能为空
   * - CREDENTIAL_CREATE_DATE_ILLEGAL
     - 100408
     - 创建日期格式非法
   * - CREDENTIAL_EXPIRE_DATE_ILLEGAL
     - 100409
     - 到期日期格式非法
   * - CREDENTIAL_CLAIM_NOT_EXISTS
     - 100410
     - Claim数据不能为空
   * - CREDENTIAL_CLAIM_DATA_ILLEGAL
     - 100411
     - Claim数据无效
   * - CREDENTIAL_ID_NOT_EXISTS
     - 100412
     - ID为空
   * - CREDENTIAL_CONTEXT_NOT_EXISTS
     - 100413
     - context为空
   * - CREDENTIAL_CPT_NOT_EXISTS
     - 100416
     - cpt不存在
   * - CREDENTIAL_WEID_DOCUMENT_ILLEGAL
     - 100417
     - WeIdentity Document为空
   * - CREDENTIAL_ISSUER_INVALID
     - 100418
     - WeIdentity DID无效
   * - CREDENTIAL_EXCEPTION_VERIFYSIGNATURE
     - 100419
     - 验证签名异常
   * - ILLEGAL_INPUT
     - 160004
     - 参数为空


**调用示例**

.. code-block:: java

   CredentialService credentialService = new CredentialServiceImpl();

   HashMap<String, Object> cptJsonSchemaData = new HashMap<String, Object>(3);
   cptJsonSchemaData.put("name", "zhang san");
   cptJsonSchemaData.put("gender", "F");
   cptJsonSchemaData.put("age", 18);

   Credential credential = new Credential();
   credential.setClaim(cptJsonSchemaData);
   credential.setContext("v1");
   credential.setCptId(175);
   credential.setIssuranceDate(1551415454745L);
   credential.setCredentialId("ce32d6d6-86e6-4740-b021-1b0d98de5400");// 系统生成
   credential.setSignature("G9cMEPhilGbu4u33IlmCoDUYncGcodfX4jdpFHIjFwltITGYN/cheS8FB4BBpKJvu3quxmacyWEUo6Zw6WmPSr8=");
   credential.setExpirationDate(1551501854695L);
   credential.setIssuer("did:weid:0xd9a2536b81e19ced12af21fb6b2a8455bbe54001");

   CredentialWrapper credentialWrapper = new CredentialWrapper();
   credentialWrapper.setCredential(credential)

   //属性是否纰漏，0:不披露 ,1:纰漏
   HashMap<String, Object> disclosure = new HashMap<String, Object>(3);
   disclosure.put("name", 1);
   disclosure.put("gender", 1);
   disclosure.put("age", 1);

   credentialWrapper.setDisclosure(disclosure);

   WeIdPublicKey weIdPublicKey = new WeIdPublicKey();
   weIdPublicKey.setPublicKey("13161444623157635919577071263152435729269604287924587017945158373362984739390835280704888860812486081963832887336483721952914804189509503053687001123007342");

   ResponseData<Boolean> response = credentialService.verifyCredentialWithSpecifiedPubKey(credentialWrapper, weIdPublicKey);


.. code-block:: text

   返回结果如：
   result: true
   errorCode: 0
   errorMessage: success

----

4. getCredentialHash
~~~~~~~~~~~~~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.CredentialService.getCredentialHash
   接口定义:ResponseData<String> getCredentialHash(Credential args)
   接口描述: 传入Credential信息生成Credential整体的Hash值。

**接口入参**\ :   com.webank.weid.protocol.base.Credential

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - context
     - String
     - Y
     - 版本
     - 默认为v1
   * - id
     - String
     - Y
     - 证书编号
     - 
   * - cptId
     - Integer
     - Y
     - cptId
     - 
   * - issuer
     - String
     - Y
     - WeIdentity DID
     - 
   * - issuranceDate
     - Long
     - Y
     - 创建日期
     - 
   * - expirationDate
     - Long
     - Y
     - 到期日期
     - 
   * - signature
     - String
     - Y
     - 签名数据
     - 
   * - claim
     - Map<String, Object>
     - Y
     - Claim数据
     - 


**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - CREDENTIAL_EXPIRED
     - 100402
     - 过期
   * - CREDENTIAL_SIGNATURE_BROKEN
     - 100405
     - 签名破坏
   * - CREDENTIAL_CREATE_DATE_ILLEGAL
     - 100408
     - 创建日期格式非法
   * - CREDENTIAL_EXPIRE_DATE_ILLEGAL
     - 100409
     - 到期日期格式非法
   * - CREDENTIAL_CLAIM_NOT_EXISTS
     - 100410
     - Claim数据不能为空
   * - CREDENTIAL_ID_NOT_EXISTS
     - 100412
     - ID为空
   * - CREDENTIAL_CONTEXT_NOT_EXISTS
     - 100413
     - context为空
   * - CREDENTIAL_CPT_NOT_EXISTS
     - 100416
     - CPT不存在
   * - CREDENTIAL_ISSUER_INVALID
     - 100418
     - WeIdentity DID无效
   * - ILLEGAL_INPUT
     - 160004
     - 参数为空


**调用示例**

.. code-block:: java

   CredentialService credentialService = new CredentialServiceImpl();

   HashMap<String, Object> claim = new HashMap<>();
   claim.put("name", "zhang san");
   claim.put("gender", "F");
   claim.put("age", "18");

   Credential credential = new Credential();
   credential.setClaim(claim);
   credential.setContext("v1");
   credential.setCptId(Integer.valueOf(155));
   credential.setExpirationDate(1551434712408L);
   credential.setId("d0859e54-0c0d-4320-9bb6-5174b86dd598");
   credential.setIssuer("did:weid:0x30404b47c6c5811d49e28ea2306c804d16618017");
   credential.setIssuranceDate(1551348312461L);
   credential.setSignature("HGBhyAwy+q5C7MjXhRssZNiVOXmGwzNIypzrleEwEZKgeOV1bpmm3v/wl0z6ACdIUGHkIJiCDlLS9NhDwVf/1nY=");

   ResponseData<String> responseHash = credentialService.getCredentialHash(credential);


.. code-block:: text

   返回结果如：
   result: 0x06e17058f71f1c352a83519fb1ac25ffd0f509a3a8c71e4bdcaea16c28ab4ba7
   errorCode: 0
   errorMessage: success


**时序图**

.. mermaid::

   sequenceDiagram
   participant 调用者
   participant CredentialService
   调用者->>CredentialService: 调用GetCredentialHash()
   CredentialService->>CredentialService: 入参非空、格式及合法性检查
   opt 入参校验失败
   CredentialService-->>调用者: 报错，提示参数不合法并退出
   end
   CredentialService->>CredentialService: 生成凭证Hash
   CredentialService-->>调用者: 返回凭证Hash

----


WeIDService
^^^^^^^^^^^

1. createWeId
~~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.WeIdService.createWeId
   接口定义:ResponseData<CreateWeIdDataResult> createWeId()
   接口描述: 内部创建公私钥，并链上注册WeIdentity DID， 并返回公钥、私钥以及WeIdentity DID。

**接口入参**\ :   无

**接口返回**\ :   com.webank.weid.protocol.response.ResponseData\<CreateWeIdDataResult>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - CreateWeIdDataResult
     - JavaBean
     - 见下


com.webank.weid.protocol.response.CreateWeIdDataResult

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - weId
     - String
     - 公钥WeIdentity DID格式字符串
     - 格式: did:weid:0x………………….
   * - userWeIdPublicKey
     - WeIdPublicKey
     - JavaBean
     - 
   * - userWeIdPrivateKey
     - WeIdPrivateKey
     - JavaBean
     - 


com.webank.weid.protocol.base.WeIdPublicKey

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - publicKey
     - String
     - 数字公钥
     - 如下调用示例返回，使用十进制数字表示


com.webank.weid.protocol.base.WeIdPrivateKey

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - privateKey
     - String
     - 数字私钥
     - 如下调用示例返回，使用十进制数字表示


**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - WEID_KEYPAIR_CREATE_FAILED
     - 10107
     - 创建密钥对失败
   * - TRANSACTION_TIMEOUT
     - 160001
     - 超时
   * - TRANSACTION_EXECUTE_ERROR
     - 160002
     - 交易错误
   * - WEID_PRIVATEKEY_DOES_NOT_MATCH
     - 100106
     - 私钥和weid不匹配
   * - UNKNOW_ERROR
     - 160003
     - 其他错误

**调用示例**

.. code-block:: java

   WeIdService weIdService = new WeIdServiceImpl();

   ResponseData<CreateWeIdDataResult> response = weIdService.createWeId();


.. code-block:: text

   输出结果如下：
   result:(com.webank.weid.protocol.response.CreateWeIdDataResult)
      weId: did:weid:0xc5ec24c2b8458960ab831217c170fd45b30fdb5f
      userWeIdPublicKey:(com.webank.weid.protocol.base.WeIdPublicKey)
         publicKey: 2219492746493166575571557478126167798854228217264776355511106640465872367922038153573407826553975483611122239014024827628219503654750946324157317248604668
      userWeIdPrivateKey:(com.webank.weid.protocol.base.WeIdPrivateKey)
         privateKey: 81348696400544184397958960012981789811325836654754093196685283522118281623132
   errorCode: 0
   errorMessage: success


**时序图**

.. mermaid::

   sequenceDiagram
   调用者->>WeIdentity SDK: 调用CreateWeID()
   WeIdentity SDK->>WeIdentity SDK: 创建公私钥对
   WeIdentity SDK->>区块链节点: 调用智能合约
   区块链节点->>区块链节点: 以事件的方式记录created属性和public key属性
   区块链节点->>区块链节点: 记录当前的最新块高
   区块链节点-->>WeIdentity SDK: 创建成功
   WeIdentity SDK-->>调用者:新创建好的WeIdentity DID以及公私钥对

----

2. createWeId
~~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.WeIdService.createWeId
   接口定义:ResponseData<String> createWeId(CreateWeIdArgs createWeIdArgs)
   接口描述: 根据传入的公私钥，链上注册WeIdentity DID，并返回WeIdentity DID。

**接口入参**\ :  com.webank.weid.protocol.request.CreateWeIdArgs

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - publicKey
     - String
     - Y
     - 数字公钥
     - 
   * - weIdPrivateKey
     - WeIdPrivateKey
     - Y
     - JavaBean
     - 后期鉴权使用


com.webank.weid.protocol.base.WeIdPrivateKey

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - privateKey
     - String
     - 数字私钥
     - 使用十进制数字表示


**接口返回**\ :   com.webank.weid.protocol.response.ResponseData\<String>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - String
     - 公钥WeIdentity DID格式字符串
     - 如：did:weid:0x………………….


**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - WEID_PUBLICKEY_AND_PRIVATEKEY_NOT_MATCHED
     - 10108
     - 公私钥不成对
   * - WEID_PRIVATEKEY_INVALID
     - 100103
     - 私钥格式非法
   * - WEID_ALREADY_EXIST
     - 100105
     - WeIdentity DID已存在
   * - WEID_PRIVATEKEY_DOES_NOT_MATCH
     - 100106
     - 私钥不与WeIdentity DID所对应
   * - TRANSACTION_TIMEOUT
     - 160001
     - 超时
   * - TRANSACTION_EXECUTE_ERROR
     - 160002
     - 交易错误
   * - ILLEGAL_INPUT
     - 160004
     - 参数为空
   * - UNKNOW_ERROR
     - 160003
     - 其他异常
   * - WEID_PUBLICKEY_INVALID
     - 100102
     - 公钥无效

**调用示例**

.. code-block:: java

   WeIdService weIdService = new WeIdServiceImpl();

   CreateWeIdArgs createWeIdArgs=new CreateWeIdArgs();
   createWeIdArgs.setPublicKey("3170586013528159122810404405684371177363772600077686241395171916056446369200124546148658301195520758036825885083801000770553553432523213872809219246853677");
        
   WeIdPrivateKey weIdPrivateKey = new WeIdPrivateKey();
   weIdPrivateKey.setPrivateKey("110459435729243083765832443137164859127043044502701844885201931614524765526930");
        
   createWeIdArgs.setWeIdPrivateKey(weIdPrivateKey);
   
   ResponseData<String> response = weIdService.createWeId(createWeIdArgs);
   BaseBean.print(response);


.. code-block:: text

   输出结果如下：
   result: did:weid:0x02ccf036610051ce2752440a8cf34f36b15ba584
   errorCode: 0
   errorMessage: success

**时序图**

.. mermaid::

   sequenceDiagram
   Note over 调用者:传入自己的WeIdentity DID及用作authentication的私钥
   调用者->>WeIdentity SDK:调用CreateWeID()
   WeIdentity SDK->>区块链节点:调用智能合约
   区块链节点->>区块链节点: 检查调用者的身份是否和WeIdentity DID匹配　　　
   opt 身份校验不通过
   区块链节点-->>WeIdentity SDK:报错，提示私钥不匹配并退出
   WeIdentity SDK-->>调用者:报错退出
   end
   区块链节点->>区块链节点 : 以事件的方式记录created属性和public key属性
   区块链节点->>区块链节点 : 记录当前的最新块高
   区块链节点-->>WeIdentity SDK: 创建成功
   WeIdentity SDK-->>调用者:新创建好的WeIdentity DID

----

3. getWeIdDocumentJson
~~~~~~~~~~~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.WeIdService.getWeIdDocumentJson
   接口定义:ResponseData<String> getWeIdDocumentJson(String weId)
   接口描述: 根据WeIdentity DID查询WeIdentity DID Document信息，并以JSON格式返回。

**接口入参**\ :   String

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - weId
     - String
     - Y
     - WeIdentity DID字符串
     - 


**接口返回**\ :   com.webank.weid.protocol.response.ResponseData\<String>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - String
     - weidDocument Json
     - 


**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - WEID_INVALID
     - 100101
     - 无效的WeIdentity DID
   * - WEID_DOES_NOT_EXIST
     - 100104
     - WeIdentity DID不存在
   * - TRANSACTION_TIMEOUT
     - 160001
     - 超时
   * - TRANSACTION_EXECUTE_ERROR
     - 160002
     - 交易错误
   * - ILLEGAL_INPUT
     - 160004
     - 参数为空


**调用示例**

.. code-block:: java

   WeIdService weIdService = new WeIdServiceImpl();

   ResponseData<String> response = weIdService.getWeIdDocumentJson("did:weid:0xc3e9d897d8cfc728625fc0227f12d67606262de5");


.. code-block:: text

   返回结果如下：
   result: {
     "@context": "https://weidentity.webank.com/did/v1",
     "id" : "did:weid:0xc3e9d897d8cfc728625fc0227f12d67606262de5",
     "created" : 1551766834422,
     "updated" : 1551766838110,
     "publicKey" : [ {
       "id" : "did:weid:0xc3e9d897d8cfc728625fc0227f12d67606262de5#keys-0",
       "type" : "Secp256k1VerificationKey2018",
       "owner" : "did:weid:0xc3e9d897d8cfc728625fc0227f12d67606262de5",
       "publicKey" : "1424477587883675440556168635073684564574790624092696341294721105203216881760356855938297063705631397760295795346770052976570043430801654973987840953244136"
     } ],
     "authentication" : [ {
       "type" : "Secp256k1SignatureAuthentication2018",
       "publicKey" : "did:weid:0xc3e9d897d8cfc728625fc0227f12d67606262de5#keys-0"
     } ],
     "service" : [ {
       "type" : "drivingCardService",
       "serviceEndpoint" : "https://weidentity.webank.com/endpoint/xxxxx"
     } ]
   }
   errorCode: 0
   errorMessage: success


**时序图**

（同时也包含getWeIDDocment时序）

.. mermaid::

   sequenceDiagram
   调用者->>WeIdentity SDK : 传入指定的WeIdentity DID
   WeIdentity SDK->>区块链节点: 调用智能合约
   区块链节点->>区块链节点: 查找记录该WeIdentity DID关联的属性事件最后一次更新时的块高
   区块链节点-->>WeIdentity SDK: 返回
   loop 解析事件
   WeIdentity SDK->>区块链节点: 根据块高，过滤该区块里的属性事件
   区块链节点-->>WeIdentity SDK: 返回
   WeIdentity SDK->>WeIdentity SDK: 根据块高，获取到对应区块所有交易
   WeIdentity SDK->>WeIdentity SDK: 根据交易获取交易回执
   WeIdentity SDK->>WeIdentity SDK: 根据交易回执过滤跟当前WeIdentity DID相关的属性事件
   WeIdentity SDK->>WeIdentity SDK: 根据不同的key，解析public key, authentication, service endpoint
   WeIdentity SDK->>WeIdentity SDK: 组装WeIdentity Document
   WeIdentity SDK->>WeIdentity SDK: 根据当前事件找到上一个事件对应的块高
   end
   WeIdentity SDK-->>调用者:返回WeIdentity Document

----

4. getWeIDDocment
~~~~~~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.WeIdService.getWeIdDocument
   接口定义:ResponseData<WeIdDocument> getWeIdDocument(String weId)
   接口描述: 根据WeIdentity DID查询出WeIdentity DID Document对象。

**接口入参**\ :  String

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - weId
     - String
     - Y
     - WeIdentity DID字符串
     - 


**接口返回**\ :   com.webank.weid.protocol.response.ResponseData\<WeIdDocument>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - WeIdDocument
     - JavaBean
     - 见下


com.webank.weid.protocol.base.WeIdDocument

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - id
     - String
     - WeIdentity DID
     - 
   * - created
     - Long
     - 创建时间
     - 
   * - updated
     - Long
     - 更新时间
     - 
   * - publicKey
     - List\ :raw-html-m2r:`<PublicKeyProperty>`
     - JavaBean
     - 列出公钥集合，见下
   * - authentication
     - List\ :raw-html-m2r:`<AuthenticationProperty>`
     - JavaBean
     - 认证方集合，见下
   * - service
     - List\ :raw-html-m2r:`<ServiceProperty>`
     - JavaBean
     - 服务端点集合，见下


com.webank.weid.protocol.base.PublicKeyProperty

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - id
     - String
     - 
     - 
   * - type
     - String
     - 类型
     - 默认为：Secp256k1VerificationKey2018
   * - owner
     - String
     - 拥有者WeIdentity DID
     - 
   * - publicKey
     - String
     - 数字公钥
     - 


com.webank.weid.protocol.base.AuthenticationProperty

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - type
     - String
     - 类型
     - 默认为：Secp256k1SignatureAuthentication2018
   * - publicKey
     - String
     - 
     - 


com.webank.weid.protocol.base.ServiceProperty

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - type
     - String
     - 类型
     - 
   * - serviceEndpoint
     - String
     - 
     - 


**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - WEID_INVALID
     - 100101
     - 无效的WeIdentity DID
   * - WEID_DOES_NOT_EXIST
     - 100104
     - WeIdentity DID不存在
   * - TRANSACTION_TIMEOUT
     - 160001
     - 超时
   * - TRANSACTION_EXECUTE_ERROR
     - 160002
     - 交易错误
   * - ILLEGAL_INPUT
     - 160004
     - 参数为空


**调用示例**

.. code-block:: java

   WeIdService weIdService = new WeIdServiceImpl();

   ResponseData<WeIdDocument> response = weIdService.getWeIdDocument("did:weid:0xe07aa06dbcc4d80885e3d4cdc75bc187790e9994");


.. code-block:: text

   返回结果如下：
   result:(com.webank.weid.protocol.base.WeIdDocument)
      id: did:weid:0xe07aa06dbcc4d80885e3d4cdc75bc187790e9994
      created: 1551853715005
      updated: 1551853719471
      publicKey:(java.util.ArrayList)
         [0]:com.webank.weid.protocol.base.PublicKeyProperty
            id: did:weid:0xe07aa06dbcc4d80885e3d4cdc75bc187790e9994#keys-0
            type: Secp256k1VerificationKey2018
            owner: did:weid:0xe07aa06dbcc4d80885e3d4cdc75bc187790e9994
            publicKey: 7966626699473840855739909549597166317960127502079511039317194665721968236964327500717279287845534595945164138995372742704212445297835796372919896301169083
      authentication:(java.util.ArrayList)
         [0]:com.webank.weid.protocol.base.AuthenticationProperty
            type: Secp256k1SignatureAuthentication2018
            publicKey: did:weid:0xe07aa06dbcc4d80885e3d4cdc75bc187790e9994#keys-0
      service:(java.util.ArrayList)
         [0]:com.webank.weid.protocol.base.ServiceProperty
            type: drivingCardService
            serviceEndpoint: https://weidentity.webank.com/endpoint/8377464
   errorCode: 0
   errorMessage: success

----

5. setPublicKey
~~~~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.WeIdService.setPublicKey
   接口定义:ResponseData<Boolean> setPublicKey(SetPublicKeyArgs setPublicKeyArgs)
   接口描述: 根据WeIdentity DID添加公钥。

**接口入参**\ :   com.webank.weid.protocol.request.SetPublicKeyArgs

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - weId
     - String
     - Y
     - WeIdentity DID格式字符串
     - 如：did:weid:1:0x....
   * - type
     - String
     - Y
     - hash套件
     - 默认：Secp256k1
   * - owner
     - String
     - N
     - 所有者
     - 默认为当前WeIdentity DID
   * - publicKey
     - String
     - Y
     - 数字公钥
     - 
   * - userWeIdPrivateKey
     - WeIdPrivateKey
     - Y
     - JavaBean
     - 交易私钥，后期鉴权使用，见下


com.webank.weid.protocol.base.WeIdPrivateKey

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - privateKey
     - String
     - 数字私钥
     - 使用十进制数字表示


**接口返回**\ :   com.webank.weid.protocol.response.ResponseData\<Boolean>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - Boolean
     - 是否set成功
     - 


**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - WEID_INVALID
     - 100101
     - 无效的WeIdentity DID
   * - WEID_PRIVATEKEY_INVALID
     - 100103
     - 私钥格式非法
   * - WEID_PRIVATEKEY_DOES_NOT_MATCH
     - 100106
     - 私钥不与WeIdentity DID所对应
   * - TRANSACTION_TIMEOUT
     - 160001
     - 超时
   * - TRANSACTION_EXECUTE_ERROR
     - 160002
     - 交易错误
   * - ILLEGAL_INPUT
     - 160004
     - 参数为空


**调用示例**

.. code-block:: java

   WeIdService weIdService = new WeIdServiceImpl();

   SetPublicKeyArgs setPublicKeyArgs=new SetPublicKeyArgs();
   setPublicKeyArgs.setWeId("did:weid:0x96f1c4ed3f4b108dfff05e6886e43a28031a79b7");
   setPublicKeyArgs.setType("Secp256k1");
   setPublicKeyArgs.setPublicKey("13161444623157635919577071263152435729269604287924587017945158373362984739390835280704888860812486081963832887336483721952914804189509503053687001123007342");

   WeIdPrivateKey weIdPrivateKey = new WeIdPrivateKey();
   weIdPrivateKey.setPrivateKey("22118377247951384085592454025144569147966463413556762198493610051879558498782");

   setPublicKeyArgs.setUserWeIdPrivateKey(weIdPrivateKey);

   ResponseData<Boolean> response = weIdService.setPublicKey(setPublicKeyArgs);


.. code-block:: text

   返回结果如下：
   result: true
   errorCode: 0
   errorMessage: success

**时序图**


.. mermaid::

   sequenceDiagram
   Note over 调用者:传入自己的WeIdentity DID及用作authentication的公私钥
   调用者->>WeIdentity SDK : 调用setPublicKey来添加公钥。
   WeIdentity SDK->>WeIdentity SDK:拿私钥来重新加载合约对象
   WeIdentity SDK->>区块链节点: 调用智能合约
   区块链节点->>区块链节点: 检查调用者的身份是否和WeIdentity DID匹配　　　
   opt 身份校验不通过
   区块链节点-->>WeIdentity SDK:报错，提示私钥不匹配并退出
   WeIdentity SDK-->>调用者:报错退出
   end
   区块链节点->>区块链节点:将公钥和WeIdentity DID以及上次记录的块高写到属性事件中
   区块链节点->>区块链节点:记录最新块高
   区块链节点-->>WeIdentity SDK:返回
   WeIdentity SDK-->>调用者:返回调用结果

----

6. setService
~~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.WeIdService.setService
   接口定义:ResponseData<Boolean> setService(SetServiceArgs setServiceArgs)
   接口描述: 根据WeIdentity DID添加Service信息。

**接口入参**\ :   com.webank.weid.protocol.request.SetServiceArgs

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - weId
     - String
     - Y
     - WeIdentity DID格式字符串
     - 如：did:weid:0x.....
   * - type
     - String
     - Y
     - 类型
     - 如：drivingCardService
   * - serviceEndpoint
     - String
     - Y
     - 服务端点
     - 如："https://weidentity.webank.com/endpoint/8377464"
   * - userWeIdPrivateKey
     - WeIdPrivateKey
     - Y
     - JavaBean
     - 交易私钥，后期鉴权使用，见下


com.webank.weid.protocol.base.WeIdPrivateKey

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - privateKey
     - String
     - 数字私钥
     - 使用十进制数字表示


**接口返回**\ :   com.webank.weid.protocol.response.ResponseData\<Boolean>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - Boolean
     - 是否set成功
     - 


**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - WEID_INVALID
     - 100101
     - 无效的WeIdentity DID
   * - WEID_PRIVATEKEY_INVALID
     - 100103
     - 私钥格式非法
   * - WEID_PRIVATEKEY_DOES_NOT_MATCH
     - 100106
     - 私钥不与WeIdentity DID所对应
   * - TRANSACTION_TIMEOUT
     - 160001
     - 超时
   * - TRANSACTION_EXECUTE_ERROR
     - 160002
     - 交易错误
   * - ILLEGAL_INPUT
     - 160004
     - 参数为空


**调用示例**

.. code-block:: java

   WeIdService weIdService = new WeIdServiceImpl();

   SetServiceArgs setServiceArgs = new SetServiceArgs();
   setServiceArgs.setWeId("did:weid:0x96f1c4ed3f4b108dfff05e6886e43a28031a79b7");
   setServiceArgs.setType("drivingCardService");
   setServiceArgs.setServiceEndpoint("https://weidentity.webank.com/endpoint/8377464");

   WeIdPrivateKey weIdPrivateKey = new WeIdPrivateKey();
   weIdPrivateKey.setPrivateKey("22118377247951384085592454025144569147966463413556762198493610051879558498782");

   setServiceArgs.setUserWeIdPrivateKey(weIdPrivateKey);

   ResponseData<Boolean> response = weIdService.setService(setServiceArgs);


.. code-block:: text

   返回结果如下：
   result: true
   errorCode: 0
   errorMessage: success

**时序图**

.. mermaid::

   sequenceDiagram
   Note over 调用者:传入自己的WeIdentity DID及要用作<br>authentication的私钥，<br>以及service endpoint
   调用者->>WeIdentity SDK : 调用setAuthentication来添加认证。
   WeIdentity SDK->>WeIdentity SDK:拿私钥来重新加载合约对象
   WeIdentity SDK->>区块链节点: 调用智能合约
   区块链节点->>区块链节点: 检查调用者的身份是否和WeIdentity DID匹配　　　
   opt 身份校验不通过
   区块链节点-->>WeIdentity SDK:报错，提示私钥不匹配并退出
   WeIdentity SDK-->>调用者:报错退出
   end
   区块链节点->>区块链节点:将service endpoint和WeIdentity DID以及上次记录的块高写到属性事件中
   区块链节点->>区块链节点:记录最新块高
   区块链节点-->>WeIdentity SDK:返回
   WeIdentity SDK-->>调用者:返回调用结果

----

7. setAuthentication
~~~~~~~~~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.WeIdService.setAuthentication
   接口定义:ResponseData<Boolean> setAuthentication(SetAuthenticationArgs setAuthenticationArgs)
   接口描述: 根据WeIdentity DID添加认证者。

**接口入参**\ :   com.webank.weid.protocol.request.SetAuthenticationArgs

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - weId
     - String
     - Y
     - WeIdentity DID格式字符串
     - 如：did:weid:0x....
   * - type
     - String
     - Y
     - hash类型
     - 
   * - owner
     - String
     - N
     - 所有者
     - 默认为当前WeIdentity DID
   * - publicKey
     - String
     - Y
     - 数字公钥
     - 
   * - userWeIdPrivateKey
     - WeIdPrivateKey
     - Y
     - JavaBean
     - 交易私钥，后期鉴权使用，见下


com.webank.weid.protocol.base.WeIdPrivateKey

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - privateKey
     - String
     - 数字私钥
     - 使用十进制数字表示


**接口返回**\ :   com.webank.weid.protocol.response.ResponseData\<Boolean>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - Boolean
     - 是否set成功
     - 


**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - WEID_INVALID
     - 100101
     - 无效的WeIdentity DID
   * - WEID_PRIVATEKEY_INVALID
     - 100103
     - 私钥格式非法
   * - WEID_PRIVATEKEY_DOES_NOT_MATCH
     - 100106
     - 私钥不与WeIdentity DID所对应
   * - TRANSACTION_TIMEOUT
     - 160001
     - 超时
   * - TRANSACTION_EXECUTE_ERROR
     - 160002
     - 交易错误
   * - ILLEGAL_INPUT
     - 160004
     - 参数为空


**调用示例**

.. code-block:: java

   WeIdService weIdService = new WeIdServiceImple();

   SetAuthenticationArgs setAuthenticationArgs=new SetAuthenticationArgs();
   setAuthenticationArgs.setWeId("did:weid:0x96f1c4ed3f4b108dfff05e6886e43a28031a79b7");
   setAuthenticationArgs.setPublicKey("13161444623157635919577071263152435729269604287924587017945158373362984739390835280704888860812486081963832887336483721952914804189509503053687001123007342");
   setAuthenticationArgs.setType("RsaSignatureAuthentication2018");
        
   WeIdPrivateKey weIdPrivateKey = new WeIdPrivateKey();
   weIdPrivateKey.setPrivateKey("22118377247951384085592454025144569147966463413556762198493610051879558498782");

   setAuthenticationArgs.setUserWeIdPrivateKey(weIdPrivateKey);
        
   ResponseData<Boolean> response = weIdService.setAuthentication(setAuthenticationArgs);


.. code-block:: text

   返回结果如下：
   result: true
   errorCode: 0
   errorMessage: success

**时序图**

.. mermaid::

   sequenceDiagram
   Note over 调用者:传入自己的WeIdentity DID及用作authentication的公私钥
   调用者->>WeIdentity SDK : 调用setAuthentication来添加认证。
   WeIdentity SDK->>WeIdentity SDK:拿私钥来重新加载合约对象
   WeIdentity SDK->>区块链节点: 调用智能合约
   区块链节点->>区块链节点: 检查调用者的身份是否和WeIdentity DID匹配　　　
   opt 身份校验不通过
   区块链节点-->>WeIdentity SDK:报错，提示私钥不匹配并退出
   WeIdentity SDK-->>调用者:报错退出
   end
   区块链节点->>区块链节点:将authentication和WeIdentity DID以及上次记录的块高写到属性事件中
   区块链节点->>区块链节点:记录最新块高
   区块链节点-->>WeIdentity SDK:返回
   WeIdentity SDK-->>调用者:返回调用结果

----

7. isWeIdExist
~~~~~~~~~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.WeIdService.isWeIdExist
   接口定义:ResponseData<Boolean> isWeIdExist(String weId)
   接口描述: 根据WeIdentity DID判断链上是否存在。
 

**接口入参**\ :   String

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - weId
     - String
     - Y
     - WeIdentity DID格式字符串
     - 如：did:weid:0x....

**接口返回**\ :   com.webank.weid.protocol.response.ResponseData\<Boolean>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - Boolean
     - 是否set成功
     - 


**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - WEID_INVALID
     - 100101
     - 无效的WeIdentity DID
   * - TRANSACTION_TIMEOUT
     - 160001
     - 超时
   * - TRANSACTION_EXECUTE_ERROR
     - 160002
     - 交易错误
   * - UNKNOW_ERROR
     - 160003
     - 未知异常


**调用示例**

.. code-block:: java

   WeIdService weIdService = new WeIdServiceImpl();
   
   ResponseData<Boolean> response = weIdService.isWeIdExist("did:weid:0x0106595955ce4713fd169bfa68e599eb99ca2e9f");


.. code-block:: text

   返回结果如下：
   result: true
   errorCode: 0
   errorMessage: success
---

**时序图**

.. mermaid::

   sequenceDiagram
         调用者->>WeIdentity SDK : 传入WeIdentity DID，调用isWeIdExist来判断是否存在。
   opt 参数校验
   Note over WeIdentity SDK:非空检查和有效性检查
   WeIdentity SDK->>WeIdentity SDK:报错，提示参数不合法并退出
   end
   WeIdentity SDK->>区块链节点: 传入WeIdentity DID链上存在性校验
         区块链节点-->>WeIdentity SDK:返回
   WeIdentity SDK-->>调用者:返回调用结果

---


EvidenceService
^^^^^^^^^^^^^^^^^

1. createEvidence
~~~~~~~~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.EvidenceService.createEvidence
   接口定义:ResponseData<String> createEvidence(Credential credential, WeIdPrivateKey weIdPrivateKey)
   接口描述: 生成凭证存证信息并上链，有判断要求数据有效，相关非空验证等。
   注意：本接口并不进行凭证的有效性验证，也就是说，上链的凭证源有可能无效。
   调用方有义务事先调用CredentialService.verifyCredential()进行判断以避免脏数据。
   传入的私钥将会成为链上存证的签名方。此签名方和凭证的Issuer可以不是同一方。

**接口入参**\ : 

com.webank.weid.protocol.base.Credential

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - context
     - String
     - Y
     - 版本
     - 默认为v1
   * - id
     - String
     - Y
     - 证书编号
     - 
   * - cptId
     - Integer
     - Y
     - cptId
     - 
   * - issuer
     - String
     - Y
     - WeIdentity DID
     - 
   * - issuranceDate
     - Long
     - Y
     - 创建日期
     - 
   * - expirationDate
     - Long
     - Y
     - 到期日期
     - 
   * - signature
     - String
     - Y
     - 签名数据
     - 
   * - claim
     - Map<String, Object>
     - Y
     - Claim数据
     - 

com.webank.weid.protocol.base.WeIdPrivateKey

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - privateKey
     - String
     - 数字私钥
     - 使用十进制数字表示

**接口返回**\ :   com.webank.weid.protocol.response.ResponseData\<String>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - String
     - 创建的凭证合约地址
     - 为空则表示失败

**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - CPT_JSON_SCHEMA_INVALID
     - 100301
     - Json Schema非法
   * - CREDENTIAL_ERROR
     - 100400
     - Credential标准错误
   * - CREDENTIAL_EXPIRED
     - 100402
     - 过期
   * - CREDENTIAL_ISSUER_MISMATCH
     - 100403
     - issuer与签名不匹配
   * - CREDENTIAL_SIGNATURE_BROKEN
     - 100405
     - 签名破坏
   * - CREDENTIAL_REVOKED
     - 100406
     - 已被撤销
   * - CREDENTIAL_ISSUER_NOT_EXISTS
     - 100407
     - WeIdentity DID不能为空
   * - CREDENTIAL_CREATE_DATE_ILLEGAL
     - 100408
     - 创建日期格式非法
   * - CREDENTIAL_EXPIRE_DATE_ILLEGAL
     - 100409
     - 到期日期格式非法
   * - CREDENTIAL_CLAIM_NOT_EXISTS
     - 100410
     - Claim数据不能为空
   * - CREDENTIAL_CLAIM_DATA_ILLEGAL
     - 100411
     - Claim数据无效
   * - CREDENTIAL_ID_NOT_EXISTS
     - 100412
     - ID为空
   * - CREDENTIAL_CONTEXT_NOT_EXISTS
     - 100413
     - context为空
   * - CREDENTIAL_CPT_NOT_EXISTS
     - 100416
     - cpt不存在
   * - CREDENTIAL_WEID_DOCUMENT_ILLEGAL
     - 100417
     - WeIdentity Document为空
   * - CREDENTIAL_EVIDENCE_BASE_ERROR
     - 100500
     - Evidence标准错误
   * - TRANSACTION_TIMEOUT
     - 160001
     - 超时
   * - TRANSACTION_EXECUTE_ERROR
     - 160002
     - 交易错误
   * - ILLEGAL_INPUT
     - 160004
     - 参数为空


**调用示例**

.. code-block:: java

   EvidenceService evidenceService = new EvidenceServiceImpl();

   HashMap<String, Object> claim = new HashMap<>();
   claim.put("name", "zhang san");
   claim.put("gender", "F");
   claim.put("age", "18");

   Credential credential = new Credential();       
   credential.setClaim(claim);
   credential.setContext("v1");
   credential.setCptId(Integer.valueOf(155));
   credential.setExpirationDate(1551434712408L);
   credential.setId("d0859e54-0c0d-4320-9bb6-5174b86dd598");
   credential.setIssuer("did:weid:0x30404b47c6c5811d49e28ea2306c804d16618017");
   credential.setIssuranceDate(1551348312461L);
   credential.setSignature("HGBhyAwy+q5C7MjXhRssZNiVOXmGwzNIypzrleEwEZKgeOV1bpmm3v/wl0z6ACdIUGHkIJiCDlLS9NhDwVf/1nY=");

   WeIdPrivateKey weIdPrivateKey = new WeIdPrivateKey();
   weIdPrivateKey.setPrivateKey("22118377247951384085592454025144569147966463413556762198493610051879558498782");

   ResponseData<String> response = evidenceService.createEvidence(credential, weIdPrivateKey);


.. code-block:: text

   返回结果如：
   result: 0x1e6b0d970fbcb727982dfe12c2efd7846bd966bd
   errorCode: 0
   errorMessage: success

**时序图**

.. mermaid::

   sequenceDiagram
   participant 调用者
   participant EvidenceService
   participant 区块链节点
   调用者->>EvidenceService: 调用CreateEvidence()
   EvidenceService->>EvidenceService: 入参非空、格式及合法性检查
   opt 入参校验失败
   EvidenceService-->>调用者: 报错，提示参数不合法并退出
   end
   EvidenceService->>EvidenceService: 生成凭证Hash
   EvidenceService->>EvidenceService: 基于凭证Hash生成签名值
   EvidenceService->>区块链节点: 调用智能合约，创建并上传凭证存证
   区块链节点-->>EvidenceService: 返回创建结果
   opt 创建失败
   EvidenceService-->>调用者: 报错并退出
   end
   EvidenceService-->>调用者: 返回成功

----

2. getEvidence
~~~~~~~~~~~~~~~~~~~


**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.EvidenceService.getEvidence
   接口定义:ResponseData<EvidenceInfo> getEvidence(String evidenceAddress)
   接口描述: 根据传入的凭证存证地址，在链上查找凭证存证信息。


**接口入参**\ :   String

**接口返回**\ :   com.webank.weid.protocol.base.EvidenceInfo;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - credentialHash
     - String
     - Y
     - 凭证Hash值
     - 是一个66个字节的字符串，以0x开头
   * - signers
     - List<String>
     - Y
     - 凭证签发者
     - 链上允许存在多个凭证签发者
   * - signatures
     - List<String>
     - Y
     - 签发者生成签名
     - 和每个签发者一一按序对应的签名值

**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - CREDENTIAL_EVIDENCE_NOT_EXISTS_ON_CHAIN
     - 100401
     - Credential入参为空
   * - CREDENTIAL_EVIDENCE_BASE_ERROR
     - 100500
     - Evidence标准错误
   * - TRANSACTION_TIMEOUT
     - 160001
     - 超时
   * - TRANSACTION_EXECUTE_ERROR
     - 160002
     - 交易错误
   * - ILLEGAL_INPUT
     - 160004
     - 参数为空


**调用示例**

.. code-block:: java

   EvidenceService evidenceService = new EvidenceServiceImpl();

   ResponseData<Evidence> response = evidenceService.getEvidence("0x1e6b0d970fbcb727982dfe12c2efd7846bd966bd");


.. code-block:: text

   返回结果如：
   result:(com.webank.weid.protocol.base.EvidenceInfo)
      credentialHash: 0x06e17058f71f1c352a83519fb1ac25ffd0f509a3a8c71e4bdcaea16c28ab4ba7
      signers:(java.util.ArrayList)
         [0]:0x96f1c4ed3f4b108dfff05e6886e43a28031a79b7
      signatures:(java.util.ArrayList)
         [0]:GySF8ijP3pIjdnyDkjBQogSSBCMsRXfgFSOKTftSZ4KjOrQjObqpBjbKNmdEG5e3+A81a8hI4kOxeQY+249gkno=
   errorCode: 0
   errorMessage: success

**时序图**

.. mermaid::

   sequenceDiagram
   participant 调用者
   participant EvidenceService
   participant 区块链节点
   调用者->>EvidenceService: 调用GetEvidence()
   EvidenceService->>EvidenceService: 入参非空、格式及合法性检查
   opt 入参校验失败
   EvidenceService-->>调用者: 报错，提示参数不合法并退出
   end
   EvidenceService->>区块链节点: 调用智能合约，查询凭证存证内容
   区块链节点-->>EvidenceService: 返回查询结果
   opt 查询出错
   EvidenceService-->>调用者: 报错并退出
   end
   EvidenceService-->>调用者: 返回成功

----

3. verify()
~~~~~~~~~~~~~~~~~~~~~

**基本信息**

.. code-block:: text

   接口名称:com.webank.weid.rpc.EvidenceService.verify
   接口定义:ResponseData<Boolean> verify(Credential credential, String evidenceAddress)
   接口描述: 根据传入的凭证和链上凭证对比，验证其是否遭到篡改。

**接口入参**\ : com.webank.weid.protocol.base.Credential

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 非空
     - 说明
     - 备注
   * - context
     - String
     - Y
     - 版本
     - 默认为v1
   * - id
     - String
     - Y
     - 证书编号
     - 
   * - cptId
     - Integer
     - Y
     - cptId
     - 
   * - issuer
     - String
     - Y
     - WeIdentity DID
     - 
   * - issuranceDate
     - Long
     - Y
     - 创建日期
     - 
   * - expirationDate
     - Long
     - Y
     - 到期日期
     - 
   * - signature
     - String
     - Y
     - 签名数据
     - 
   * - claim
     - Map<String, Object>
     - Y
     - Claim数据
     - 

String：以地址形式存在的String，会进行入参检查

**接口返回**\ :   com.webank.weid.protocol.response.ResponseData\<Boolean>;

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 说明
     - 备注
   * - errorCode
     - Integer
     - 返回结果码
     - 
   * - errorMessage
     - String
     - 返回结果描述
     - 
   * - result
     - Boolean
     - 是否set成功
     - 

**此方法返回code**

.. list-table::
   :header-rows: 1

   * - enum
     - code
     - desc
   * - SUCCESS
     - 0
     - 成功
   * - CPT_JSON_SCHEMA_INVALID
     - 100301
     - Json Schema非法
   * - CREDENTIAL_ERROR
     - 100400
     - Credential标准错误
   * - CREDENTIAL_EXPIRED
     - 100402
     - 过期
   * - CREDENTIAL_ISSUER_MISMATCH
     - 100403
     - issuer与签名不匹配
   * - CREDENTIAL_SIGNATURE_BROKEN
     - 100405
     - 签名破坏
   * - CREDENTIAL_REVOKED
     - 100406
     - 已被撤销
   * - CREDENTIAL_ISSUER_NOT_EXISTS
     - 100407
     - WeIdentity DID不能为空
   * - CREDENTIAL_CREATE_DATE_ILLEGAL
     - 100408
     - 创建日期格式非法
   * - CREDENTIAL_EXPIRE_DATE_ILLEGAL
     - 100409
     - 到期日期格式非法
   * - CREDENTIAL_CLAIM_NOT_EXISTS
     - 100410
     - Claim数据不能为空
   * - CREDENTIAL_CLAIM_DATA_ILLEGAL
     - 100411
     - Claim数据无效
   * - CREDENTIAL_ID_NOT_EXISTS
     - 100412
     - ID为空
   * - CREDENTIAL_CONTEXT_NOT_EXISTS
     - 100413
     - context为空
   * - CREDENTIAL_CPT_NOT_EXISTS
     - 100416
     - cpt不存在
   * - CREDENTIAL_WEID_DOCUMENT_ILLEGAL
     - 100417
     - WeIdentity Document为空
   * - CREDENTIAL_EVIDENCE_BASE_ERROR
     - 100500
     - Evidence标准错误
   * - CREDENTIAL_EVIDENCE_HASH_MISMATCH
     - 100501
     - Evidence Hash不匹配
   * - CREDENTIAL_EVIDENCE_ID_MISMATCH
     - 100502
     - Evidence ID不匹配
   * - TRANSACTION_TIMEOUT
     - 160001
     - 超时
   * - TRANSACTION_EXECUTE_ERROR
     - 160002
     - 交易错误
   * - ILLEGAL_INPUT
     - 160004
     - 参数为空
   * - CREDENTIAL_EVIDENCE_CONTRACT_FAILURE_ALREADY_EXISTS
     - 500401
     - Evidence ID已存在
   * - CREDENTIAL_EVIDENCE_CONTRACT_FAILURE_NO_PERMISSION
     - 500402
     - Evidence操作无权限


**调用示例**

.. code-block:: java

   EvidenceService evidenceService = new EvidenceServiceImpl();

   HashMap<String, Object> claim = new HashMap<>();
   claim.put("name", "zhang san");
   claim.put("gender", "F");
   claim.put("age", "18");

   Credential credential = new Credential();
   credential.setClaim(claim);
   credential.setContext("v1");
   credential.setCptId(Integer.valueOf(155));
   credential.setExpirationDate(1551434712408L);
   credential.setId("d0859e54-0c0d-4320-9bb6-5174b86dd598");
   credential.setIssuer("did:weid:0x30404b47c6c5811d49e28ea2306c804d16618017");
   credential.setIssuranceDate(1551348312461L);
   credential.setSignature("HGBhyAwy+q5C7MjXhRssZNiVOXmGwzNIypzrleEwEZKgeOV1bpmm3v/wl0z6ACdIUGHkIJiCDlLS9NhDwVf/1nY=");

   String evidenceAddress = "0x1e6b0d970fbcb727982dfe12c2efd7846bd966bd";

   ResponseData<Boolean> response = evidenceService.verify(credential, evidenceAddress);

   
.. code-block:: text

   返回结果如：
   result: true
   errorCode: 0
   errorMessage: success


**时序图**

.. mermaid::

   sequenceDiagram
   participant 调用者
   participant EvidenceService
   participant WeIdService
   participant 区块链节点
   调用者->>EvidenceService: 调用VerifyEvidence()
   EvidenceService->>EvidenceService: 入参非空、格式及合法性检查
   opt 入参校验失败
   EvidenceService-->>调用者: 报错，提示参数不合法并退出
   end
   EvidenceService->>EvidenceService: 调用GetEvidence()查询凭证内容
   EvidenceService->>区块链节点: 调用智能合约，查询凭证存证内容
   区块链节点-->>EvidenceService: 返回查询结果
   opt 查询出错
   EvidenceService-->>调用者: 返回验证失败，报错并退出
   end
   EvidenceService->>EvidenceService: 生成凭证Hash，与链上凭证Hash对比是否一致
   opt Hash不一致
   EvidenceService-->>调用者: 返回验证失败，报错并退出
   end
   EvidenceService->>WeIdService: 根据存证中签名方信息，调用GetWeIdDocument()查询WeID公钥
   WeIdService->>区块链节点: 调用智能合约，查询WeID公钥
   区块链节点-->>WeIdService: 返回查询结果
   EvidenceService->>EvidenceService: 验证存证中签名是否为与凭证Hash一致
   opt 验签失败
   EvidenceService-->>调用者: 返回验证失败，报错并退出
   end
   EvidenceService-->>调用者: 返回验证成功
