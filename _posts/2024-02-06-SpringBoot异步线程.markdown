---
layout: post
title:  "SpringBoot异步线程"
datetime:   2024-02-06 10:27:26
categories: SpringBoot异步线程
---
# SpringBoot异步线程

配置

```java
package com.ydzb.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.Executor;
import java.util.concurrent.ThreadPoolExecutor;

/**
 * @author lth
 * @date 2023/10/30
 */
@Configuration
@EnableAsync
public class AsyncConfig {
    private static final int CORE_POOL_SIZE = 4;
    private static final int MAX_POOL_SIZE = 8;
    private static final int QUEUE_CAPACITY = 200;
    private static final int KEEP_ALIVE_SECONDS = 60;
    private static final String THREAD_NAME_PREFIX = "CREW-Task-ThreadPool-";

    @Bean("logThreadPoolTaskExecutor")
    public Executor logThreadPoolTaskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        // 核心线程数量
        executor.setCorePoolSize(CORE_POOL_SIZE);
        // 最大线程数量
        executor.setMaxPoolSize(MAX_POOL_SIZE);
        // 队列中最大任务数
        executor.setQueueCapacity(QUEUE_CAPACITY);
        // 线程名称前缀
        executor.setThreadNamePrefix(THREAD_NAME_PREFIX);
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.DiscardOldestPolicy());
        // 线程空闲后最大存活时间
        executor.setKeepAliveSeconds(KEEP_ALIVE_SECONDS);
        // 初始化线程池
        executor.initialize();
        return executor;
    }
}

```

