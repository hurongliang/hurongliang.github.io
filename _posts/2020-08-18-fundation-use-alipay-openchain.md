---
layout: post
title: 项目实践：公益项目接入区块链
categories: 蚂蚁开放联盟链 区块链 存证
excerpt: java项目如何快速接入区块链技术，蚂蚁金服开放联盟链如何落地？
---

# 项目背景

某大型公益组织打算建设一个公益平台，为了增加平台的公信力，打算引入区块链技术，将用户的捐赠行为存入区块链网络。

# 技术选型

有两个方案可供选择，一是部署私有的同盟链，好处是可以减少外部依赖，缺点是开发和维护成本较高，同时私有联盟链的公信力不如开放链。二是使用市场上成熟的区块链服务，有点是开发维护成本低，缺点是需要持续付费。

国内的区块链服务其实都不是真正去中心化的，比如蚂蚁区块链，是有20个节点（后续可能会扩展）充电授信网络节点。在这些节点内是去中心化的，但是这20个节点仍然受人为控制（比如将他们关掉）。

因为项目不缺钱，需要尽快发布。于是决定使用目前较为成熟的蚂蚁金服开放联盟链，我们对其进行了调研，结合项目本身，得出以下结论：

- 蚂蚁金服最近才发布开放联盟链，项目处于初期阶段，缺少社区支持，影响接入效率。
- 产品功能和配套工具处于迭代中，接入方后续可能需要跟进调整。
- 不在区块链中存储状态数据，不适用合约服务，只用存证服务。
- 服务调用需要消耗燃料，因此进入区块链的存证数据应该尽量的少。

之所以选择蚂蚁金服开放联盟链，一是没有更好的选择，二是有蚂蚁金服的信用和技术背书。但在实际接入过程中还是存在一些问题。

# 接入指南

官方提供可接入文档，地址：<https://tech.antfin.com/docs/2/143566>

这里给出重点步骤。

一、开通开放联盟链。需要用到支付宝账号，建议使用项目专用账号。地址：<https://openchain.cloud.alipay.com/>

二、购买燃料，调用区块链服务时需要消耗燃料。地址：<https://openchain.cloud.alipay.com/balance>

三、创建链账户，建议选择秘钥托管，可以减少后续SDK接入的配置，不过我们项目使用的秘钥非托管。地址：<https://openchain.cloud.alipay.com/account/lists>

四、给链账户分配燃料，通常每次存证服务需要消耗3w燃料，根据自身业务量来分配，本项目测试阶段分配了500W的燃料。地址：<https://openchain.cloud.alipay.com/balance>

五、SDK接入

因为本项目服务端使用springboot架构，因此需要用到他们提供的Java SDK。官方接入文档地址：<https://tech.antfin.com/docs/2/146924>

这里遇到的第一个坑是官方依赖没有发布到maven仓库中心，因此需要将sdk作为本地jar包引入项目。pom.xml配置参考

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    ...

    <dependencies>

        ...

        <!-- blockchain -->
        <dependency>
            <groupId>org.springframework.retry</groupId>
            <artifactId>spring-retry</artifactId>
            <version>1.2.4.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>com.antfinancial.baas</groupId>
            <artifactId>mychain-rest-lib</artifactId>
            <version>0.10.2.11</version>
            <scope>system</scope>
            <systemPath>${project.basedir}/lib/rest-client-2.11-2020.03.12-async-SNAPSHOT-with-dependencies.jar
            </systemPath>
        </dependency>
    </dependencies>

    <build>
        <resources>
            <resource>
                <directory>${project.basedir}/lib</directory>
                <targetPath>BOOT-INF/lib/</targetPath>
                <includes>
                    <include>*.jar</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <targetPath>BOOT-INF/classes/</targetPath>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <includeSystemScope>true</includeSystemScope>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.5</version>
                <configuration>
                    <skipTests>true</skipTests>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

第二个坑，官方的SDK是个胖SDK，将SDK依赖也一并打包进去了，里面包含spring boot 1.5.9.RELEASE版本，而我们的项目使用springboot 2.1.9，类冲突导致项目无法启动。

这里有几种解决思路：

- 1. 将区块链服务使用springboot 1.5.9 单独部署，但是我们项目使用了spring cloud集群，出了要接入springboot还要接入eureka，不适合本项目。

- 2. 使用HTTP方式接入，需要很多额外的操作，增加了开发成本，违背快速上线的项目目标，不适合本项目。

- 3. 修改jar包，移除springboot相关依赖，重新打包。经过测试，sdk能正常运行，本项目采用的这个方法。
  
  六、调用存证服务将数据上链

参照官方教程调用SDK即可。地址：<https://tech.antfin.com/docs/2/146924#h4-u5B58u8BC1>

这里给出本项目使用的Client封装类

```java
package com.zhiyu.guangda.support.business;

import com.alibaba.fastjson.JSON;
import com.antfinancial.mychain.baas.tool.restclient.RestClient;
import com.antfinancial.mychain.baas.tool.restclient.RestClientProperties;
import com.antfinancial.mychain.baas.tool.restclient.model.ClientParam;
import com.antfinancial.mychain.baas.tool.restclient.model.Method;
import com.antfinancial.mychain.baas.tool.restclient.response.BaseResp;
import com.zhiyu.guangda.base.exception.RetryException;
import com.zhiyu.guangda.support.bo.BTQueryTransactionResponseData;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.bouncycastle.util.encoders.Base64;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.retry.annotation.Backoff;
import org.springframework.retry.annotation.Retryable;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class BlockchainClient {

    @Autowired
    private RestClient restClient;

    @Autowired
    private RestClientProperties restClientProperties;

    /**
     * 存证上链
     *
     * @param data 存证内容
     * @return
     * @throws Exception
     */
    public String deposit(String data) throws Exception {
        ClientParam clientParam = restClient.createDepositTransaction(restClientProperties.getDefaultAccount(), data, 50000L);
        BaseResp resp = restClient.chainCall(clientParam.getHash(), clientParam.getSignData(), Method.DEPOSIT);
        checkResp(resp);
        log.info("存证上链成功，内容 = {}, hash = {}", data, clientParam.getHash());
        return clientParam.getHash();
    }

    /**
     * 查询存证结果
     *
     * @param hash
     * @return
     */
    public String queryTransaction(String hash) {
        BTQueryTransactionResponseData data = queryTransactionOrigin(hash);
        String transactionData = new String(Base64.decode(data.getTransactionDO().getData()));
        log.info("存证内容 {}", transactionData);
        return transactionData;
    }

    /**
     * 查询存证原始结果
     *
     * @param hash
     * @return
     */
    @Retryable(value = RetryException.class, maxAttempts = 2, backoff = @Backoff(delay = 30000, multiplier = 1))
    public BTQueryTransactionResponseData queryTransactionOrigin(String hash) {
        BaseResp resp;
        try {
            resp = restClient.chainCall(hash, null, Method.QUERYTRANSACTION);
        } catch (Exception e) {
            throw RetryException.create(e.getMessage(), e);
        }
        if (!resp.isSuccess() || !StringUtils.equals(resp.getCode(), "200")) {
            throw new RetryException(JSON.toJSONString(resp));
        }

        log.info("查询到存证结果, resp = {}, hash = {}", JSON.toJSONString(resp), hash);
        return JSON.parseObject(resp.getData(), BTQueryTransactionResponseData.class);
    }


    private void checkResp(BaseResp resp) throws Exception {
        if (!resp.isSuccess() || !StringUtils.equals(resp.getCode(), "200")) {
            throw new Exception(JSON.toJSONString(resp));
        }
    }
}
```

# 工程源码

参见：<https://github.com/hurongliang/fundation-use-alipay-openchain>

# 小结

通过该项目，我们学到了区块链技术在实际应用中如何产生价值。

早期市场培育存在过度神秘化现象，导致业界对其价值落地产生怀疑，而区块链的理念本身是有价值的，相信随着炒作热情消退，市场趋于理性，我们会找到更多区块链技术与现实需求相结合的落地场景。

在技术演进和更大规模落地的进程中如何能否保持其去权威、公开、平等的技术理念依然存在较大不确定性。
