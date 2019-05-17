title: POST 缓存方案
date: 2019-05-15T02:47:20.649Z
tags: []
categories: []
---
## 背景

当前应用中，一方面是有国外服务器，导致客户端请求数据慢，loading时间过长；另一方面是客户端页面跳转导致的多次频繁请求，很是影响用户体验和影响服务器性能，所以需要对客户端的请求做缓存处理，以便达到减轻服务器压力（减少重复请求次数）和对用户的友好体验（添加数据库缓存）的目的。

## 目标
1. 减轻服务器压力
2. 提高用户体验

## 方案
1. 针对每个接口设置不同的缓存策略（缓存时间）
2. 请求数据时先查询数据库，有数据就返回界面，同时网络请求，数据回来返回界面，更新数据库？
3. 请求数据先查询数据库，判断是否过期，没过期直接返回界面，否则请求网络，再刷新界面和更新数据库
4. 没有网络就直接返回数据库数据。

## 根据接口细化

#### 完全不需要缓存
1. sendAddress (创建钱包)
2. sendPushInfo, unsubscribePushInfo, updatePusheToken(推送相关信息)
3. getVerifyCode, versifyPhoneAndGetInviteCode, JsUrl, verifyTicket, verifyInviteCode(验证码信息相关)
4. createNewId, restoreProfile, updateProfile,updateCellphone (创建，恢复，更新等接口)
5. becomeCandidate, reapplyCandidate, quiteCandidate, updateNodeInfo(对节点动作的操作结果，只返回基础成功与否的接口)
6. 退票，加减仓相关

#### 按照条件添加永久缓存
1. 红包详情
3. 交易详情


#### 较长时间缓存，服务端可做的缓存处理接口（规则等信息）
1. fetchUpdateInfo (App更新信息)
2. fetchThirdAppInfo, getHepDapp (Dapp 详细信息和Dapp 列表信息)
3. getNewIdCondition, getVoteCondition, getNodeCondition，getRedPacketCondition

#### 一天内时间的缓存
1. 获取挖矿信息，奖励token信息，奖励 nf 信息等
2. 轮次信息
3. 获取全网的nf

#### 要最新的数据，同时也要缓存的（短暂缓存，防止用户频繁刷新）
1. 获取用户的ProfileInfo
2. 获取公告信息 getAnnouncement
3. 红包的收发记录
4. 交易记录

## 技术调研
1. RxCache:

    使用注解方式对接口设置缓存策略，目前测试支持GET请求，（别人有参数一样的POST请求，但是自己测试post请求是缓存了，但是没有读取），缓存的结果到一个文件里面，使用LruDiskCache 算法处理缓存文件夹大小，还可以定制接口是否强制使用缓存，如果能忽略参数获取缓存结果这个方案还不错。
    
2. OkHttp3 设置 CacheInterceptor:

    手动处理请求流程，在请求过程中，根据设置好的缓存策略和有无网络情况读取数据库或者网络数据，如果有缓存返回缓存数据，同时去网络请求，回来结果再返回数据，不过前台界面不好控制数据来源..
    
