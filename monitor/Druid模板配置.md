# druid模板配置

系统内部需引入两个文件
```java
package com.my.prometheus;


import com.alibaba.druid.pool.DruidDataSource;
import io.micrometer.core.instrument.Gauge;
import io.micrometer.core.instrument.MeterRegistry;

import java.util.List;
import java.util.function.ToDoubleFunction;

class DruidCollector {

    private static final String LABEL_NAME = "pool";

    private final List<DruidDataSource> dataSources;

    private final MeterRegistry registry;

    DruidCollector(List<DruidDataSource> dataSources, MeterRegistry registry) {
        this.registry = registry;
        this.dataSources = dataSources;
    }

    void register() {
        this.dataSources.forEach((druidDataSource) -> {
            // basic configurations
            createGauge(druidDataSource, "druid_initial_size", "Initial size", (datasource) -> (double) druidDataSource.getInitialSize());
            createGauge(druidDataSource, "druid_min_idle", "Min idle", datasource -> (double) druidDataSource.getMinIdle());
            createGauge(druidDataSource, "druid_max_active", "Max active", datasource -> (double) druidDataSource.getMaxActive());

            // connection pool core metrics
            createGauge(druidDataSource, "druid_active_count", "Active count", datasource -> (double) druidDataSource.getActiveCount());
            createGauge(druidDataSource, "druid_active_peak", "Active peak", datasource -> (double) druidDataSource.getActivePeak());
            createGauge(druidDataSource, "druid_pooling_peak", "Pooling peak", datasource -> (double) druidDataSource.getPoolingPeak());
            createGauge(druidDataSource, "druid_pooling_count", "Pooling count", datasource -> (double) druidDataSource.getPoolingCount());
            createGauge(druidDataSource, "druid_wait_thread_count", "Wait thread count", datasource -> (double) druidDataSource.getWaitThreadCount());

            // connection pool detail metrics
            createGauge(druidDataSource, "druid_not_empty_wait_count", "Not empty wait count", datasource -> (double) druidDataSource.getNotEmptyWaitCount());
            createGauge(druidDataSource, "druid_not_empty_wait_millis", "Not empty wait millis", datasource -> (double) druidDataSource.getNotEmptyWaitMillis());
            createGauge(druidDataSource, "druid_not_empty_thread_count", "Not empty thread count", datasource -> (double) druidDataSource.getNotEmptyWaitThreadCount());

            createGauge(druidDataSource, "druid_logic_connect_count", "Logic connect count", datasource -> (double) druidDataSource.getConnectCount());
            createGauge(druidDataSource, "druid_logic_close_count", "Logic close count", datasource -> (double) druidDataSource.getCloseCount());
            createGauge(druidDataSource, "druid_logic_connect_error_count", "Logic connect error count", datasource -> (double) druidDataSource.getConnectErrorCount());
            createGauge(druidDataSource, "druid_physical_connect_count", "Physical connect count", datasource -> (double) druidDataSource.getCreateCount());
            createGauge(druidDataSource, "druid_physical_close_count", "Physical close count", datasource -> (double) druidDataSource.getDestroyCount());
            createGauge(druidDataSource, "druid_physical_connect_error_count", "Physical connect error count", datasource -> (double) druidDataSource.getCreateErrorCount());

            // sql execution core metrics
            createGauge(druidDataSource, "druid_error_count", "Error count", datasource -> (double) druidDataSource.getErrorCount());
            createGauge(druidDataSource, "druid_execute_count", "Execute count", datasource -> (double) druidDataSource.getExecuteCount());
            // transaction metrics
            createGauge(druidDataSource, "druid_start_transaction_count", "Start transaction count", datasource -> (double) druidDataSource.getStartTransactionCount());
            createGauge(druidDataSource, "druid_commit_count", "Commit count", datasource -> (double) druidDataSource.getCommitCount());
            createGauge(druidDataSource, "druid_rollback_count", "Rollback count", datasource -> (double) druidDataSource.getRollbackCount());

            // sql execution detail
            createGauge(druidDataSource, "druid_prepared_statement_open_count", "Prepared statement open count", datasource -> (double) druidDataSource.getPreparedStatementCount());
            createGauge(druidDataSource, "druid_prepared_statement_closed_count", "Prepared statement closed count", datasource -> (double) druidDataSource.getClosedPreparedStatementCount());
            createGauge(druidDataSource, "druid_ps_cache_access_count", "PS cache access count", datasource -> (double) druidDataSource.getCachedPreparedStatementAccessCount());
            createGauge(druidDataSource, "druid_ps_cache_hit_count", "PS cache hit count", datasource -> (double) druidDataSource.getCachedPreparedStatementHitCount());
            createGauge(druidDataSource, "druid_ps_cache_miss_count", "PS cache miss count", datasource -> (double) druidDataSource.getCachedPreparedStatementMissCount());
            createGauge(druidDataSource, "druid_execute_query_count", "Execute query count", datasource -> (double) druidDataSource.getExecuteQueryCount());
            createGauge(druidDataSource, "druid_execute_update_count", "Execute update count", datasource -> (double) druidDataSource.getExecuteUpdateCount());
            createGauge(druidDataSource, "druid_execute_batch_count", "Execute batch count", datasource -> (double) druidDataSource.getExecuteBatchCount());

            // none core metrics, some are static configurations
            createGauge(druidDataSource, "druid_max_wait", "Max wait", datasource -> (double) druidDataSource.getMaxWait());
            createGauge(druidDataSource, "druid_max_wait_thread_count", "Max wait thread count", datasource -> (double) druidDataSource.getMaxWaitThreadCount());
            createGauge(druidDataSource, "druid_login_timeout", "Login timeout", datasource -> (double) druidDataSource.getLoginTimeout());
            createGauge(druidDataSource, "druid_query_timeout", "Query timeout", datasource -> (double) druidDataSource.getQueryTimeout());
            createGauge(druidDataSource, "druid_transaction_query_timeout", "Transaction query timeout", datasource -> (double) druidDataSource.getTransactionQueryTimeout());
        });
    }

    private void createGauge(DruidDataSource weakRef, String metric, String help, ToDoubleFunction<DruidDataSource> measure) {
        Gauge.builder(metric, weakRef, measure)
                .description(help)
                .tag(LABEL_NAME, weakRef.getName())
                .register(this.registry);
    }
}
```

```java
package com.my.prometheus;


import com.alibaba.druid.pool.DruidDataSource;
import io.micrometer.core.instrument.MeterRegistry;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;

@Configuration
@ConditionalOnClass({DruidDataSource.class, MeterRegistry.class})
@Slf4j
public class DruidMetricsConfiguration {

    private final MeterRegistry registry;

    public DruidMetricsConfiguration(MeterRegistry registry) {
        this.registry = registry;
    }

    @Autowired
    public void bindMetricsRegistryToDruidDataSources(Collection<DataSource> dataSources) throws SQLException {
        List<DruidDataSource> druidDataSources = new ArrayList<>(dataSources.size());
        for (DataSource dataSource : dataSources) {
            DruidDataSource druidDataSource = dataSource.unwrap(DruidDataSource.class);
            if (druidDataSource != null) {
                druidDataSources.add(druidDataSource);
            }
        }
        DruidCollector druidCollector = new DruidCollector(druidDataSources, registry);
        druidCollector.register();
        log.info("finish register metrics to micrometer");
    }
}

```

- 假如是多数据源的情况下，需要考虑更改配置内容（具体没有实现）

## grafana官网上下载druid的模板，上面的文件也可从官网下载  

- 链接：https://grafana.com/grafana/dashboards/

- 配置方式与springboot监控模板的配置方式一致