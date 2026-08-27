```
package com.example.salary.config;

import com.example.salary.model.IndexTest;  
import org.springframework.batch.core.configuration.annotation.StepScope;  
import org.springframework.batch.core.job.Job;  
import org.springframework.batch.core.job.builder.JobBuilder;  
import org.springframework.batch.core.partition.Partitioner;  
import org.springframework.batch.core.repository.JobRepository;  
import org.springframework.batch.core.step.Step;  
import org.springframework.batch.core.step.builder.StepBuilder;  
import org.springframework.batch.infrastructure.item.ExecutionContext;  
import org.springframework.batch.infrastructure.item.ItemWriter;  
import org.springframework.batch.infrastructure.item.database.JdbcBatchItemWriter;  
import org.springframework.batch.infrastructure.item.database.JdbcCursorItemReader;  
import org.springframework.batch.infrastructure.item.database.JdbcPagingItemReader;  
import org.springframework.batch.infrastructure.item.database.Order;  
import org.springframework.batch.infrastructure.item.database.builder.JdbcBatchItemWriterBuilder;  
import org.springframework.batch.infrastructure.item.database.builder.JdbcCursorItemReaderBuilder;  
import org.springframework.batch.infrastructure.item.database.builder.JdbcPagingItemReaderBuilder;  
import org.springframework.beans.factory.annotation.Value;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
import org.springframework.jdbc.core.JdbcTemplate;  
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;  
import org.springframework.transaction.PlatformTransactionManager;

import javax.sql.DataSource;  
import java.sql.PreparedStatement;  
import java.util.ArrayList;  
import java.util.HashMap;  
import java.util.List;  
import java.util.Map;

//@Configuration  
//public class BatchConfig {  
//  
// @Bean  
// public JdbcCursorItemReader<IndexTest> reader(DataSource dataSource) {  
//  
// return new JdbcCursorItemReaderBuilder<IndexTest>()  
// .name("indexTestReader")  
// .dataSource(dataSource)  
// .sql("""  
// SELECT id, name, salary  
// FROM index_test  
// ORDER BY id  
// """)  
// .rowMapper((rs, rowNum) ->  
// new IndexTest(  
// rs.getLong("id"),  
// rs.getString("name"),  
// rs.getBigDecimal("salary")  
// ))  
// .build();  
// }  
//  
// @Bean  
// public ItemWriter<IndexTest> writer(JdbcTemplate jdbcTemplate) {  
//  
// return chunk -> {  
// long start = System.currentTimeMillis();  
// List<IndexTest> items = new ArrayList<>(chunk.getItems());  
//  
// jdbcTemplate.batchUpdate(  
// """  
// UPDATE index_test  
// SET salary = 0  
// WHERE id = ?  
// """,  
// items,  
// items.size(),  
// (PreparedStatement ps, IndexTest item) ->  
// ps.setLong(1, item.getId())  
// );  
// long end = System.currentTimeMillis();  
// System.out.println(  
// "Writer processed " + items.size()  
// + " items in " + (end - start) + " ms"  
// );  
// };  
// }  
//  
// @Bean  
// public Step salaryStep(  
// JobRepository jobRepository,  
// PlatformTransactionManager transactionManager,  
// JdbcCursorItemReader<IndexTest> reader,  
// ItemWriter<IndexTest> writer) {  
//  
// return new StepBuilder("salaryStep", jobRepository)  
// .<IndexTest, IndexTest>chunk(1000)  
// .transactionManager(transactionManager)  
// .reader(reader)  
// .writer(writer)  
// .build();  
// }  
//  
// @Bean  
// public Job salaryJob(  
// JobRepository jobRepository,  
// Step salaryStep) {  
//  
// return new JobBuilder("salaryJob", jobRepository)  
// .start(salaryStep)  
// .build();  
// }  
//}

@Configuration  
public class BatchConfig {

    // --------------------------------------------------
    // READER    // --------------------------------------------------
    @Bean
    @StepScope    public JdbcPagingItemReader<IndexTest> reader(
            DataSource dataSource,
            @Value("#{stepExecutionContext['minId']}") Long minId,
            @Value("#{stepExecutionContext['maxId']}") Long maxId) throws Exception {

        Map<String, Order> sortKeys = new HashMap<>();
        sortKeys.put("id", Order.ASCENDING);

        return new JdbcPagingItemReaderBuilder<IndexTest>()
                .name("indexTestReader")
                .dataSource(dataSource)
                .pageSize(1000)
                .selectClause("SELECT id, name, salary")
                .fromClause("FROM index_test")
                .whereClause("WHERE id BETWEEN :minId AND :maxId")
                .parameterValues(Map.of(
                        "minId", minId,
                        "maxId", maxId
                ))
                .sortKeys(sortKeys)
                .rowMapper((rs, rowNum) -> new IndexTest(
                        rs.getLong("id"),
                        rs.getString("name"),
                        rs.getBigDecimal("salary")
                ))
                .build();
    }


    // --------------------------------------------------
    // WRITER    // --------------------------------------------------
    @Bean
    public ItemWriter<IndexTest> writer(JdbcTemplate jdbcTemplate) {

        return chunk -> {
            long start = System.currentTimeMillis();
            jdbcTemplate.batchUpdate(
                    """
                    UPDATE index_test                    SET salary = 0                    WHERE id = ?                    """,
                    chunk.getItems(),
                    chunk.size(),
                    (ps, item) -> ps.setLong(1, item.getId())
            );
            long end = System.currentTimeMillis();

// System.out.println(  
// "Writer processed items in " + (end - start) + " ms"  
// );  
 };  
 }

    // --------------------------------------------------
    // WORKER STEP    // --------------------------------------------------
    @Bean
    public Step workerStep(
            JobRepository jobRepository,
            PlatformTransactionManager transactionManager,
            JdbcPagingItemReader<IndexTest> reader,
            ItemWriter<IndexTest> writer) {

        return new StepBuilder("workerStep", jobRepository)
                .<IndexTest, IndexTest>chunk(1000)
                .transactionManager(transactionManager)
                .reader(reader)
                .writer(writer)
                .build();
    }


    // --------------------------------------------------
    // PARTITIONER    // --------------------------------------------------
    @Bean
    public Partitioner partitioner() {

        return gridSize -> {

            Map<String, ExecutionContext> partitions = new HashMap<>();

            long totalRows = 1_000_000;
            long partitionSize = totalRows / gridSize;

            for (int i = 0; i < gridSize; i++) {

                long minId = i * partitionSize + 1;

                long maxId = (i == gridSize - 1)
                        ? totalRows
                        : (i + 1) * partitionSize;

                ExecutionContext context = new ExecutionContext();

                context.putLong("minId", minId);
                context.putLong("maxId", maxId);

                partitions.put("partition-" + i, context);
            }

            return partitions;
        };
    }


    // --------------------------------------------------
    // TASK EXECUTOR    // --------------------------------------------------
    @Bean
    public ThreadPoolTaskExecutor taskExecutor() {

        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();

        executor.setCorePoolSize(4);
        executor.setMaxPoolSize(4);
        executor.setQueueCapacity(100);

        executor.setThreadNamePrefix("partition-");

        executor.initialize();

        return executor;
    }


    // --------------------------------------------------
    // PARTITION STEP    // --------------------------------------------------
    @Bean
    public Step partitionStep(
            JobRepository jobRepository,
            Step workerStep,
            Partitioner partitioner,
            ThreadPoolTaskExecutor taskExecutor) {

        return new StepBuilder("partitionStep", jobRepository)
                .partitioner("workerStep", partitioner)
                .step(workerStep)
                .gridSize(16)
                .taskExecutor(taskExecutor)
                .build();
    }


    // --------------------------------------------------
    // JOB    // --------------------------------------------------
    @Bean
    public Job salaryJob(
            JobRepository jobRepository,
            Step partitionStep) {

        return new JobBuilder("salaryJob", jobRepository)
                .start(partitionStep)
                .build();
    }

}

```