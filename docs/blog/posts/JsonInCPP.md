---
title: nlohmann json和rapidjson
authors: ["Misaki"]
date: 2025-02-17
categories: ["Json"]
---

在c++开发过程中不免要用到Json数据格式来传递，所以简单聊聊常用的两个Json库，并且他们也都较好的支持跨平台开发。

<!-- more -->

## Nlohmann Json

[:octicons-link-16: 文档: https://json.nlohmann.me/](https://json.nlohmann.me/)

nlohmann json提供了header only的版本，从集成角度考虑是及其方便处理的。

使用上符合modern c++语法（毕竟标题就是这个），几乎和用STL差不多，开发人员一致好评的丝滑。

它也支持从STL的任意序列容器初始化获得json对象，所以相当适合用来处理数据反序列化到JSON字符串的过程。

那么代价是什么呢？就是牺牲了一定的性能，尤其是当对象中存储了大量字符串时（例如二进制文件的base64字符串），处理耗时会明显增加。对于时间不敏感的服务来说，能支撑快速高效的开发工作已经是相当优势了。

## rapidjson

[:octicons-link-16: 文档: https://rapidjson.org/](https://rapidjson.org/)

由腾讯提供的json库，同样是header only的。

以高性能著称，独立且内存友好。对Unicode有良好的支持，如果服务需要处理各类字符，那么它绝对是优先考虑的库。

不过性能的代价就是使用体验上大打折扣，每个操作你都需要自己管理allocator。你也不想看到处理不当（尤其是初次使用的开发者）导致丢失数据吧？

文档也更加传统，不如上文提及的nlohmann json来的好上手。

## 对比

当json字符串相对较大时，rapidjson的耗时可以是nlohmann json的1/4，性能差距明显。

如果服务开发对时间还不算特别敏感，又希望有一些性能提升，恰好是输入大输出小的情况的话，你也能看见两个库同时出现在一个模块里的情况。

序列化交给rapidjson，反序列化交给nlohmann json。