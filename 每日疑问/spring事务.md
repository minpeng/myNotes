## spring事务详解



### 1.spring事务猜想 

> 使用切面再方法执行前开启事务，执行完提交事务，如果有异常，则回滚事务

```mysql
## Mysql事务流程

##1.开启事务
begin; 
##2.提交事务
commit;
##3.如果有异常需要回滚事务
rollback；
```

```java
// jdbc使用事务流程

//1.获取连接
Connection conn = DataSourceUtils.getConnection();
//2.开启事务
conn.setAutoCommit(false);
try {
    //3.执行业务sql操作
   	doSomething(conn);
    //4.提交事务
    conn.commit();
}catch (Exception e) {
    //回滚事务
    conn.rollback();
}finally {
    conn.close();
}
```



#### 2.spring注解@transactional 

```JAVA
public @interface Transactional {
    //当在配置文件中有多个 TransactionManager , 可以用该属性指定选择哪个事务管理器
    @AliasFor("transactionManager")
    String value() default "";

    @AliasFor("value")
    String transactionManager() default "";
	//事务传播机制
    Propagation propagation() default Propagation.REQUIRED;
	//事务隔离级别
    Isolation isolation() default Isolation.DEFAULT;
	//超时时间
    //如果超过该时间限制但事务还没有完成，则自动回滚事务。
    int timeout() default -1;
	//事务只读
    boolean readOnly() default false;
	//用于指定能够触发事务回滚的异常类型，如果有多个异常类型需要指定，各类型之间可以通过逗号分隔
    Class<? extends Throwable>[] rollbackFor() default {};

    String[] rollbackForClassName() default {};
	//抛出 no-rollback-for 指定的异常类型，不回滚事务。
    Class<? extends Throwable>[] noRollbackFor() default {};

    String[] noRollbackForClassName() default {};
}
```



#### 3.spring执行@transactional 

>  一个事务方法执行流程

+ 获取事务属性

+ 获取事务管理器

+ 获取需要事务的方法名称/获取该方法上事务的信息

+ 目标方法执行

+ 清除事务信息

+ 事务回滚

+ 事务提交

  

> 事务执行方法TransactionAspectSupport

```JAVA
public abstract class TransactionAspectSupport implements BeanFactoryAware, InitializingBean {
   @Nullable
    protected Object invokeWithinTransaction(Method method, @Nullable Class<?> targetClass, TransactionAspectSupport.InvocationCallback invocation) throws Throwable {
        // 读取事务的属性和设置
        TransactionAttributeSource tas = this.getTransactionAttributeSource();
        TransactionAttribute txAttr = tas != null ? tas.getTransactionAttribute(method, targetClass) : null;
        // 获取 beanFactory 中的 transactionManager
        PlatformTransactionManager tm = this.determineTransactionManager(txAttr);
        String joinpointIdentification = this.methodIdentification(method, targetClass, txAttr);
        Object result;
        //编程式事务（需要加入处理事务逻辑，需要显示调用事务方法）
        if (txAttr != null && tm instanceof CallbackPreferringPlatformTransactionManager) {
            TransactionAspectSupport.ThrowableHolder throwableHolder = new TransactionAspectSupport.ThrowableHolder();

            try {
                result = ((CallbackPreferringPlatformTransactionManager)tm).execute(txAttr, (status) -> {
                   
                    TransactionAspectSupport.TransactionInfo txInfo = this.prepareTransactionInfo(tm, txAttr, joinpointIdentification, status);

                    Object var9;
                    try {
                        //目标方法执行
                        Object var8 = invocation.proceedWithInvocation();
                        return var8;
                    } catch (Throwable var13) {
                        if (txAttr.rollbackOn(var13)) {
                            if (var13 instanceof RuntimeException) {
                                throw (RuntimeException)var13;
                            }

                            throw new TransactionAspectSupport.ThrowableHolderException(var13);
                        }

                        throwableHolder.throwable = var13;
                        var9 = null;
                    } finally {
                        //清除事务信息
                        this.cleanupTransactionInfo(txInfo);
                    }

                    return var9;
                });
                if (throwableHolder.throwable != null) {
                    throw throwableHolder.throwable;
                } else {
                    return result;
                }
            } catch (TransactionAspectSupport.ThrowableHolderException var19) {
                throw var19.getCause();
            } catch (TransactionSystemException var20) {
                if (throwableHolder.throwable != null) {
                    this.logger.error("Application exception overridden by commit exception", throwableHolder.throwable);
                    var20.initApplicationException(throwableHolder.throwable);
                }

                throw var20;
            } catch (Throwable var21) {
                if (throwableHolder.throwable != null) {
                    this.logger.error("Application exception overridden by commit exception", throwableHolder.throwable);
                }

                throw var21;
            }
        } else {
            // 声明式事务
            // 构建事务相关信息
            TransactionAspectSupport.TransactionInfo txInfo = this.createTransactionIfNecessary(tm, txAttr, joinpointIdentification);
            result = null;

            try {
                result = invocation.proceedWithInvocation();
            } catch (Throwable var17) {
               // 如果出现异常，则进行回滚
                this.completeTransactionAfterThrowing(txInfo, var17);
                throw var17;
            } finally {
                this.cleanupTransactionInfo(txInfo);
            }
			 // 这里通过事务处理器来对事务进行提交
            this.commitTransactionAfterReturning(txInfo);
            return result;
        }
    } 
}

```



### 4.spring事务常量
```java
public interface TransactionDefinition {
	//传播性
    
	//如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。默认为这个
    int PROPAGATION_REQUIRED = 0;
    
    //支持当前事务，如果当前没有事务，就以非事务方式执行
    int PROPAGATION_SUPPORTS = 1;
    
    //使用当前的事务，如果当前没有事务，就抛出异常。
    int PROPAGATION_MANDATORY = 2;
    
	//新建事务，如果当前存在事务，把当前事务挂起。
    int PROPAGATION_REQUIRES_NEW = 3;
    
    //以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
    int PROPAGATION_NOT_SUPPORTED = 4;
    
    //以非事务方式执行，如果当前存在事务，则抛出异常。
    int PROPAGATION_NEVER = 5;
    
    //如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作
    int PROPAGATION_NESTED = 6;
    
    //隔离级别
    //使用数据库默认的隔离级别-可重复读的
    int ISOLATION_DEFAULT = -1;
    //允许读取尚未提交的更改。可能导致脏读、幻影读或不可重复读
    int ISOLATION_READ_UNCOMMITTED = 1;
    //允许从已经提交的并发事务读取。可防止脏读，但幻影读和不可重复读仍可能会发生。
    int ISOLATION_READ_COMMITTED = 2;
    //对相同字段的多次读取的结果是一致的，除非数据被当前事务本身改变。可防止脏读和不可重复读，但幻影读仍可能发生
    int ISOLATION_REPEATABLE_READ = 4;
    //完全服从ACID的隔离级别，确保不发生脏读、不可重复读和幻影读。这在所有隔离级别中也是最慢的，因为它通常是通过完全锁定当前事务所涉及的数据表来完成的。
    int ISOLATION_SERIALIZABLE = 8;
    }

```



###  5. spring事务管理接口

```java

//平台事务管理接口
public interface PlatformTransactionManager {
    //获取事务状态
    TransactionStatus getTransaction(@Nullable TransactionDefinition var1) throws TransactionException;
	//提交事务
    void commit(TransactionStatus var1) throws TransactionException;
	//回滚事务
    void rollback(TransactionStatus var1) throws TransactionException;
}
```



### 6.事务状态TransactionStatus

```java
public class DefaultTransactionStatus extends AbstractTransactionStatus {
   
    @Nullable
    // 事务连接器
    private final Object transaction;
    // 是否是新事务
    private final boolean newTransaction;
    // 是否开启 事务同步器
    private final boolean newSynchronization;
    // 这个事务是否是 readOnly
    private final boolean readOnly;
    //是否debug模式
    private final boolean debug;
    @Nullable
    //事务是否需要挂起
    private final Object suspendedResources;
}
```



### 7.spring事务管理实现类 -AbstractPlatformTransactionManager

+ getTransaction()方法

```java
public final TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException {
        Object transaction = this.doGetTransaction();
        boolean debugEnabled = this.logger.isDebugEnabled();
        if (definition == null) {
            definition = new DefaultTransactionDefinition();
        }
		// 如果当前已经存在事务
        if (this.isExistingTransaction(transaction)) {
            // 根据不同传播机制不同处理
            return this.handleExistingTransaction((TransactionDefinition)definition, transaction, debugEnabled);
        } else if (((TransactionDefinition)definition).getTimeout() < -1) {
            // 超时不能小于-1
            throw new InvalidTimeoutException("Invalid transaction timeout", ((TransactionDefinition)definition).getTimeout());
        } else if (((TransactionDefinition)definition).getPropagationBehavior() == 2) {
           // 当前不存在事务，传播机制=MANDATORY（支持当前事务，没事务报错），报错
            throw new IllegalTransactionStateException("No existing transaction found for transaction marked with propagation 'mandatory'");
        } 
    //// 当前不存在事务，传播机制=REQUIRED/REQUIRED_NEW/NESTED,这三种情况，需要新开启事务，且加上事务同步
    else if (((TransactionDefinition)definition).getPropagationBehavior() != 0 && ((TransactionDefinition)definition).getPropagationBehavior() != 3 && ((TransactionDefinition)definition).getPropagationBehavior() != 6) {
        //事务隔离级别不是默认的
            if (((TransactionDefinition)definition).getIsolationLevel() != -1 && this.logger.isWarnEnabled()) {
                this.logger.warn("Custom isolation level specified but no actual transaction initiated; isolation level will effectively be ignored: " + definition);
            }
			//是否需要新开启同步
            boolean newSynchronization = this.getTransactionSynchronization() == 0;
            return this.prepareTransactionStatus((TransactionDefinition)definition, (Object)null, true, newSynchronization, debugEnabled, (Object)null);
        } else {
            AbstractPlatformTransactionManager.SuspendedResourcesHolder suspendedResources = this.suspend((Object)null);
            if (debugEnabled) {
                this.logger.debug("Creating new transaction with name [" + ((TransactionDefinition)definition).getName() + "]: " + definition);
            }

            try {
                
                //public static final int SYNCHRONIZATION_NEVER = 2;
 				//2的含义：永不开启同步              
                boolean newSynchronization = this.getTransactionSynchronization() != 2;
                DefaultTransactionStatus status = this.newTransactionStatus((TransactionDefinition)definition, transaction, true, newSynchronization, debugEnabled, suspendedResources);
                // 开启新事务
                this.doBegin(transaction, (TransactionDefinition)definition);
                //预备同步
                this.prepareSynchronization(status, (TransactionDefinition)definition);
                return status;
            } catch (Error | RuntimeException var7) {
                this.resume((Object)null, suspendedResources);
                throw var7;
            }
        }
    }
```



+ handleExistingTransaction存在事务之后处理方法(根据事务传播级别)

```java
private TransactionStatus handleExistingTransaction(TransactionDefinition definition, Object transaction, boolean debugEnabled) throws TransactionException {
    // 如果当前已经存在事务, 且当前事务的传播属性设置为 PROPAGATION_NEVER（5）, 那么抛出异常    
    if (definition.getPropagationBehavior() == 5) {
            throw new IllegalTransactionStateException("Existing transaction found for transaction marked with propagation 'never'");
        } else {
            AbstractPlatformTransactionManager.SuspendedResourcesHolder suspendedResources;
            boolean newSynchronization;
        
        // 如果当前事务的配置属性是 PROPAGATION_NOT_SUPPORTED（4）, 同时当前线程已经存在事务了, 那么将事务挂起
            if (definition.getPropagationBehavior() == 4) {
                if (debugEnabled) {
                    this.logger.debug("Suspending current transaction");
                }
				// 将事务的挂起
                suspendedResources = this.suspend(transaction);
                // 是否开启一个新的事务同步器
                newSynchronization = this.getTransactionSynchronization() == 0;
                //// 意味着事务方法不需要放在事务环境中执行, 同时挂起事务的信息保存在 TransactionStatus 中, 用ThreadLocal来记录
                return this.prepareTransactionStatus(definition, (Object)null, false, newSynchronization, debugEnabled, suspendedResources);
            } else if (definition.getPropagationBehavior() == 3) {
                // 如果当前事务的配置属性是 PROPAGATION_REQUIRES_NEW（3）, 创建新事务, 同时将当前线程存在的事务挂起, 与创建全新事务的过程类是, 区别在于在创建全新事务时不用考虑已有事务的挂起, 但在这里, 需要考虑已有事务的挂起
                if (debugEnabled) {
                    this.logger.debug("Suspending current transaction, creating new transaction with name [" + definition.getName() + "]");
                }
				//挂起事务
                suspendedResources = this.suspend(transaction);

                try {
                    // 是否开启一个新的事务同步器
                    newSynchronization = this.getTransactionSynchronization() != 2;
                    //  挂起事务的信息记录保存在 TransactionStatus 中,
                    DefaultTransactionStatus status = this.newTransactionStatus(definition, transaction, true, newSynchronization, debugEnabled, suspendedResources);
                    // 构造 transaction, 包括设置 ConnectionHolder, 隔离级别, timeout, 如果是新连接, 绑定到当前线程
                    this.doBegin(transaction, definition);
                    // 新同步事务的设置, 针对当前线程的设置
                    this.prepareSynchronization(status, definition);
                    return status;
                } catch (Error | RuntimeException var7) {
                    // 抛出异常，恢复刚才挂起的事务
                    this.resumeAfterBeginException(transaction, suspendedResources, var7);
                    throw var7;
                }
            } else {
               
                boolean newSynchronization;
                // 如果当前事务的配置属性是 PROPAGATION_NOT_SUPPORTED（6）嵌套事务，创建 TransactionStatus, 创建保存点
                if (definition.getPropagationBehavior() == 6) {
                    if (!this.isNestedTransactionAllowed()) {
                        throw new NestedTransactionNotSupportedException("Transaction manager does not allow nested transactions by default - specify 'nestedTransactionAllowed' property with value 'true'");
                    } else {
                        if (debugEnabled) {
                            this.logger.debug("Creating nested transaction with name [" + definition.getName() + "]");
                        }
						// 如果没有可以使用保存点的方式控制事务回滚, 那么在嵌套式事务的建立初始建立保存点
                        if (this.useSavepointForNestedTransaction()) {
                            // 在 Spring 管理的事务中, 创建事务保存点
                            DefaultTransactionStatus status = this.prepareTransactionStatus(definition, transaction, false, false, debugEnabled, (Object)null);
                            status.createAndHoldSavepoint();
                            return status;
                        } else 
                            // 有些情况是不能使用保存点操作, 比如 JTA, 那么就建立新事务
                            newSynchronization = this.getTransactionSynchronization() != 2;							
                            DefaultTransactionStatus status = this.newTransactionStatus(definition, transaction, true, newSynchronization, debugEnabled, (Object)null);
                            this.doBegin(transaction, definition);
                            this.prepareSynchronization(status, definition);
                            return status;
                        }
                    }
                } else {
                    if (debugEnabled) {
                        this.logger.debug("Participating in existing transaction");
                    }
					// 对已经存在的事务的属性进行校验
                    if (this.isValidateExistingTransaction()) {
                        // 隔离级别的校验,TransactionDefinition 与 TransactionSynchronizationManager 中的值是否一致
                        if (definition.getIsolationLevel() != -1) {
                            Integer currentIsolationLevel = TransactionSynchronizationManager.getCurrentTransactionIsolationLevel();
                            if (currentIsolationLevel == null || currentIsolationLevel != definition.getIsolationLevel()) {
                                Constants isoConstants = DefaultTransactionDefinition.constants;
                                throw new IllegalTransactionStateException("Participating transaction with definition [" + definition + "] specifies isolation level which is incompatible with existing transaction: " + (currentIsolationLevel != null ? isoConstants.toCode(currentIsolationLevel, "ISOLATION_") : "(unknown)"));
                            }
                        }
						// readOnly 的校验 
                        if (!definition.isReadOnly() && TransactionSynchronizationManager.isCurrentTransactionReadOnly()) {
                            throw new IllegalTransactionStateException("Participating transaction with definition [" + definition + "] is not marked as read-only but existing transaction is");
                        }
                    }

                    newSynchronization = this.getTransactionSynchronization() != 2;
                   // 返回 TransactionStatus 注意第三个参数 false 表示 当前事务没有使用新事务
                    return this.prepareTransactionStatus(definition, transaction, false, newSynchronization, debugEnabled, (Object)null);
                }
            }
        }
    }
```



+ 提交事务方法

```java
 private void processCommit(DefaultTransactionStatus status) throws TransactionException {
        try {
            boolean beforeCompletionInvoked = false;

            try {
                boolean unexpectedRollback = false;
                this.prepareForCommit(status);
                this.triggerBeforeCommit(status);
                this.triggerBeforeCompletion(status);
                beforeCompletionInvoked = true;
                
                // 如果有保存点（嵌套事务），清除保存点信息
                if (status.hasSavepoint()) {
                    if (status.isDebug()) {
                        this.logger.debug("Releasing transaction savepoint");
                    }

                    unexpectedRollback = status.isGlobalRollbackOnly();
                    //清除保存点信息
                    status.releaseHeldSavepoint();
                } else if (status.isNewTransaction()) {
                    // 若是新事务, 则直接提交
                    if (status.isDebug()) {
                        this.logger.debug("Initiating transaction commit");
                    }

                    unexpectedRollback = status.isGlobalRollbackOnly();
                    //提交
                    this.doCommit(status);
                } else if (this.isFailEarlyOnGlobalRollbackOnly()) {
                    unexpectedRollback = status.isGlobalRollbackOnly();
                }

                if (unexpectedRollback) {
                    throw new UnexpectedRollbackException("Transaction silently rolled back because it has been marked as rollback-only");
                }
            } catch (UnexpectedRollbackException var17) {
                this.triggerAfterCompletion(status, 1);
                throw var17;
            } catch (TransactionException var18) {
                if (this.isRollbackOnCommitFailure()) {
                    //回滚
                    this.doRollbackOnCommitException(status, var18);
                } else {
                    this.triggerAfterCompletion(status, 2);
                }

                throw var18;
            } catch (Error | RuntimeException var19) {
                if (!beforeCompletionInvoked) {
                    this.triggerBeforeCompletion(status);
                }

                this.doRollbackOnCommitException(status, var19);
                throw var19;
            }

            try {
                this.triggerAfterCommit(status);
            } finally {
                this.triggerAfterCompletion(status, 0);
            }
        } finally {
            this.cleanupAfterCompletion(status);
        }

    }
```



### 8.spring事务管理模板方法



```JAVA
//模板方法
public class TransactionTemplate extends DefaultTransactionDefinition implements TransactionOperations, InitializingBean {
    
       @Nullable
    public <T> T execute(TransactionCallback<T> action) throws TransactionException {
        Assert.state(this.transactionManager != null, "No PlatformTransactionManager set");
        if (this.transactionManager instanceof CallbackPreferringPlatformTransactionManager) {
            return ((CallbackPreferringPlatformTransactionManager)this.transactionManager).execute(this, action);
        } else {
            //1.获取事务状态
            TransactionStatus status = this.transactionManager.getTransaction(this);

            Object result;
            try {
                 // 2.执行业务逻辑
                result = action.doInTransaction(status);
            } catch (Error | RuntimeException var5) {
                ////事务回滚
                this.rollbackOnException(status, var5);
                throw var5;
            } catch (Throwable var6) {
                //事务回滚
                this.rollbackOnException(status, var6);
                throw new UndeclaredThrowableException(var6, "TransactionCallback threw undeclared checked exception");
            }
			//事务提交
            this.transactionManager.commit(status);
            return result;
        }
    }
    
}

```



### 参考链接

> <https://www.javazhiyin.com/35501.html>
>
> <https://www.jianshu.com/p/1bfa61868823>
>
> <https://www.cnblogs.com/dennyzhangdd/p/9602673.html>