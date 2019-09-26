---
title: Spring Cloud 注册中心笔记
description:  pring Cloud 注册中心笔记
date: 2018-08-25 21:43:00
tags: 
    - Spring 
    - Spring Cloud
categories:
    - Spring
    - Spring Cloud
---

# 注册中心源码 EnableErurkaServer

## 注册 注册中心启动后,会生成一个注册接口为ApplicationResource#addInstance 方法为post

ApplicationResource#addInstance 注册接口
```java
    @POST
    @Consumes({"application/json", "application/xml"})
    public Response addInstance(InstanceInfo info, @HeaderParam("x-netflix-discovery-replication") String isReplication) {
        logger.debug("Registering instance {} (replication={})", info.getId(), isReplication);
        if (this.isBlank(info.getId())) {
            return Response.status(400).entity("Missing instanceId").build();
        } else if (this.isBlank(info.getHostName())) {
            return Response.status(400).entity("Missing hostname").build();
        } else if (this.isBlank(info.getIPAddr())) {
            return Response.status(400).entity("Missing ip address").build();
        } else if (this.isBlank(info.getAppName())) {
            return Response.status(400).entity("Missing appName").build();
        } else if (!this.appName.equals(info.getAppName())) {
            return Response.status(400).entity("Mismatched appName, expecting " + this.appName + " but was " + info.getAppName()).build();
        } else if (info.getDataCenterInfo() == null) {
            return Response.status(400).entity("Missing dataCenterInfo").build();
        } else if (info.getDataCenterInfo().getName() == null) {
            return Response.status(400).entity("Missing dataCenterInfo Name").build();
        } else {
            DataCenterInfo dataCenterInfo = info.getDataCenterInfo();
            if (dataCenterInfo instanceof UniqueIdentifier) {
                String dataCenterInfoId = ((UniqueIdentifier)dataCenterInfo).getId();
                if (this.isBlank(dataCenterInfoId)) {
                    boolean experimental = "true".equalsIgnoreCase(this.serverConfig.getExperimental("registration.validation.dataCenterInfoId"));
                    if (experimental) {
                        String entity = "DataCenterInfo of type " + dataCenterInfo.getClass() + " must contain a valid id";
                        return Response.status(400).entity(entity).build();
                    }

                    if (dataCenterInfo instanceof AmazonInfo) {
                        AmazonInfo amazonInfo = (AmazonInfo)dataCenterInfo;
                        String effectiveId = amazonInfo.get(MetaDataKey.instanceId);
                        if (effectiveId == null) {
                            amazonInfo.getMetadata().put(MetaDataKey.instanceId.getName(), info.getId());
                        }
                    } else {
                        logger.warn("Registering DataCenterInfo of type {} without an appropriate id", dataCenterInfo.getClass());
                    }
                }
            }

            //PeerAwareInstanceRegistry注册方法
            this.registry.register(info, "true".equals(isReplication)); 
            return Response.status(204).build();
        }
    }
```
PeerAwareInstanceRegistry#register()
```java

    public void register(InstanceInfo info, boolean isReplication) {
        int leaseDuration = 90;
        if (info.getLeaseInfo() != null && info.getLeaseInfo().getDurationInSecs() > 0) {
            leaseDuration = info.getLeaseInfo().getDurationInSecs();
        }
        //使用父类注册方法
        super.register(info, leaseDuration, isReplication); 
        this.replicateToPeers(PeerAwareInstanceRegistryImpl.Action.Register, info.getAppName(), info.getId(), info, (InstanceStatus)null, isReplication);
    }
```
AbstractInstanceRegistry#register()

```java
    public void register(InstanceInfo registrant, int leaseDuration, boolean isReplication) {
        try {
            this.read.lock();
            Map<String, Lease<InstanceInfo>> gMap = (Map)this.registry.get(registrant.getAppName());
            EurekaMonitors.REGISTER.increment(isReplication);
            if (gMap == null) {
                ConcurrentHashMap<String, Lease<InstanceInfo>> gNewMap = new ConcurrentHashMap();
                gMap = (Map)this.registry.putIfAbsent(registrant.getAppName(), gNewMap);
                if (gMap == null) {
                    gMap = gNewMap;
                }
            }

            Lease<InstanceInfo> existingLease = (Lease)((Map)gMap).get(registrant.getId());
            if (existingLease != null && existingLease.getHolder() != null) {
                Long existingLastDirtyTimestamp = ((InstanceInfo)existingLease.getHolder()).getLastDirtyTimestamp();
                Long registrationLastDirtyTimestamp = registrant.getLastDirtyTimestamp();
                logger.debug("Existing lease found (existing={}, provided={}", existingLastDirtyTimestamp, registrationLastDirtyTimestamp);
                if (existingLastDirtyTimestamp > registrationLastDirtyTimestamp) {
                    logger.warn("There is an existing lease and the existing lease's dirty timestamp {} is greater than the one that is being registered {}", existingLastDirtyTimestamp, registrationLastDirtyTimestamp);
                    logger.warn("Using the existing instanceInfo instead of the new instanceInfo as the registrant");
                    registrant = (InstanceInfo)existingLease.getHolder();
                }
            } else {
                Object var6 = this.lock;
                synchronized(this.lock) {
                    if (this.expectedNumberOfClientsSendingRenews > 0) {
                        ++this.expectedNumberOfClientsSendingRenews;
                        this.updateRenewsPerMinThreshold();
                    }
                }

                logger.debug("No previous lease information found; it is new registration");
            }

            Lease<InstanceInfo> lease = new Lease(registrant, leaseDuration);
            if (existingLease != null) {
                lease.setServiceUpTimestamp(existingLease.getServiceUpTimestamp());
            }

            ((Map)gMap).put(registrant.getId(), lease);
            AbstractInstanceRegistry.CircularQueue var20 = this.recentRegisteredQueue;
            synchronized(this.recentRegisteredQueue) {
                this.recentRegisteredQueue.add(new Pair(System.currentTimeMillis(), registrant.getAppName() + "(" + registrant.getId() + ")"));
            }

            if (!InstanceStatus.UNKNOWN.equals(registrant.getOverriddenStatus())) {
                logger.debug("Found overridden status {} for instance {}. Checking to see if needs to be add to the overrides", registrant.getOverriddenStatus(), registrant.getId());
                if (!this.overriddenInstanceStatusMap.containsKey(registrant.getId())) {
                    logger.info("Not found overridden id {} and hence adding it", registrant.getId());
                    this.overriddenInstanceStatusMap.put(registrant.getId(), registrant.getOverriddenStatus());
                }
            }

            InstanceStatus overriddenStatusFromMap = (InstanceStatus)this.overriddenInstanceStatusMap.get(registrant.getId());
            if (overriddenStatusFromMap != null) {
                logger.info("Storing overridden status {} from map", overriddenStatusFromMap);
                registrant.setOverriddenStatus(overriddenStatusFromMap);
            }

            InstanceStatus overriddenInstanceStatus = this.getOverriddenInstanceStatus(registrant, existingLease, isReplication);
            registrant.setStatusWithoutDirty(overriddenInstanceStatus);
            if (InstanceStatus.UP.equals(registrant.getStatus())) {
                lease.serviceUp();
            }

            registrant.setActionType(ActionType.ADDED);
            this.recentlyChangedQueue.add(new AbstractInstanceRegistry.RecentlyChangedItem(lease));
            registrant.setLastUpdatedTimestamp();
            this.invalidateCache(registrant.getAppName(), registrant.getVIPAddress(), registrant.getSecureVipAddress());
            logger.info("Registered instance {}/{} with status {} (replication={})", new Object[]{registrant.getAppName(), registrant.getId(), registrant.getStatus(), isReplication});
        } finally {
            this.read.unlock();
        }

    }

```

## 续约 
InstanceResource 续约接口 method put
```java
    @PUT
    public Response renewLease(@HeaderParam("x-netflix-discovery-replication") String isReplication, @QueryParam("overriddenstatus") String overriddenStatus, @QueryParam("status") String status, @QueryParam("lastDirtyTimestamp") String lastDirtyTimestamp) {
        boolean isFromReplicaNode = "true".equals(isReplication);
        boolean isSuccess = this.registry.renew(this.app.getName(), this.id, isFromReplicaNode);
        if (!isSuccess) {
            logger.warn("Not Found (Renew): {} - {}", this.app.getName(), this.id);
            return Response.status(Status.NOT_FOUND).build();
        } else {
            Response response;
            if (lastDirtyTimestamp != null && this.serverConfig.shouldSyncWhenTimestampDiffers()) {
                response = this.validateDirtyTimestamp(Long.valueOf(lastDirtyTimestamp), isFromReplicaNode);
                if (response.getStatus() == Status.NOT_FOUND.getStatusCode() && overriddenStatus != null && !InstanceStatus.UNKNOWN.name().equals(overriddenStatus) && isFromReplicaNode) {
                    this.registry.storeOverriddenStatusIfRequired(this.app.getAppName(), this.id, InstanceStatus.valueOf(overriddenStatus));
                }
            } else {
                response = Response.ok().build();
            }

            logger.debug("Found (Renew): {} - {}; reply status={}", new Object[]{this.app.getName(), this.id, response.getStatus()});
            return response;
        }
    }
```
通过以上源码得知注册中心本质上是一个双层的 ConcurrentHashMap，存储在内存中的。

第一层的 key 是spring.application.name，value 是第二层 ConcurrentHashMap；
第二层 ConcurrentHashMap 的 key 是服务的 InstanceId，value 是 Lease 对象；
Lease 对象包含了服务详情和服务治理相关的属性。

# 服务端源码 EnableEurekaClient

注解EnableEurekaClient 通过Spring boot自动配置后会生成一DiscoveryClient实例
DiscoveryClient 初始化时 拉取注册表信息，服务注册，初始化心跳机制，缓存刷新，按需注册定时任务等等。

```java
@Inject
DiscoveryClient(ApplicationInfoManager applicationInfoManager, EurekaClientConfig config, AbstractDiscoveryClientOptionalArgs args, Provider<BackupRegistry> backupRegistryProvider) {
        this.RECONCILE_HASH_CODES_MISMATCH = Monitors.newCounter("DiscoveryClient_ReconcileHashCodeMismatch");
        this.FETCH_REGISTRY_TIMER = Monitors.newTimer("DiscoveryClient_FetchRegistry");
        this.REREGISTER_COUNTER = Monitors.newCounter("DiscoveryClient_Reregister");
        this.localRegionApps = new AtomicReference();
        this.fetchRegistryUpdateLock = new ReentrantLock();
        this.healthCheckHandlerRef = new AtomicReference();
        this.remoteRegionVsApps = new ConcurrentHashMap();
        this.lastRemoteInstanceStatus = InstanceStatus.UNKNOWN;
        this.eventListeners = new CopyOnWriteArraySet();
        this.registrySize = 0;
        this.lastSuccessfulRegistryFetchTimestamp = -1L;
        this.lastSuccessfulHeartbeatTimestamp = -1L;
        this.isShutdown = new AtomicBoolean(false);
        if (args != null) {
            this.healthCheckHandlerProvider = args.healthCheckHandlerProvider;
            this.healthCheckCallbackProvider = args.healthCheckCallbackProvider;
            this.eventListeners.addAll(args.getEventListeners());
            this.preRegistrationHandler = args.preRegistrationHandler;
        } else {
            this.healthCheckCallbackProvider = null;
            this.healthCheckHandlerProvider = null;
            this.preRegistrationHandler = null;
        }

        this.applicationInfoManager = applicationInfoManager;
        InstanceInfo myInfo = applicationInfoManager.getInfo();
        this.clientConfig = config;
        staticClientConfig = this.clientConfig;
        this.transportConfig = config.getTransportConfig();
        this.instanceInfo = myInfo;
        if (myInfo != null) {
            this.appPathIdentifier = this.instanceInfo.getAppName() + "/" + this.instanceInfo.getId();
        } else {
            logger.warn("Setting instanceInfo to a passed in null value");
        }

        this.backupRegistryProvider = backupRegistryProvider;
        this.urlRandomizer = new InstanceInfoBasedUrlRandomizer(this.instanceInfo);
        this.localRegionApps.set(new Applications());
        this.fetchRegistryGeneration = new AtomicLong(0L);
        this.remoteRegionsToFetch = new AtomicReference(this.clientConfig.fetchRegistryForRemoteRegions());
        this.remoteRegionsRef = new AtomicReference(this.remoteRegionsToFetch.get() == null ? null : ((String)this.remoteRegionsToFetch.get()).split(","));
        if (config.shouldFetchRegistry()) {
            this.registryStalenessMonitor = new ThresholdLevelsMetric(this, "eurekaClient.registry.lastUpdateSec_", new long[]{15L, 30L, 60L, 120L, 240L, 480L});
        } else {
            this.registryStalenessMonitor = ThresholdLevelsMetric.NO_OP_METRIC;
        }

        if (config.shouldRegisterWithEureka()) {
            this.heartbeatStalenessMonitor = new ThresholdLevelsMetric(this, "eurekaClient.registration.lastHeartbeatSec_", new long[]{15L, 30L, 60L, 120L, 240L, 480L});
        } else {
            this.heartbeatStalenessMonitor = ThresholdLevelsMetric.NO_OP_METRIC;
        }

        logger.info("Initializing Eureka in region {}", this.clientConfig.getRegion());
        if (!config.shouldRegisterWithEureka() && !config.shouldFetchRegistry()) {
            logger.info("Client configured to neither register nor query for data.");
            this.scheduler = null;
            this.heartbeatExecutor = null;
            this.cacheRefreshExecutor = null;
            this.eurekaTransport = null;
            this.instanceRegionChecker = new InstanceRegionChecker(new PropertyBasedAzToRegionMapper(config), this.clientConfig.getRegion());
            DiscoveryManager.getInstance().setDiscoveryClient(this);
            DiscoveryManager.getInstance().setEurekaClientConfig(config);
            this.initTimestampMs = System.currentTimeMillis();
            logger.info("Discovery Client initialized at timestamp {} with initial instances count: {}", this.initTimestampMs, this.getApplications().size());
        } else {
            try {
                this.scheduler = Executors.newScheduledThreadPool(2, (new ThreadFactoryBuilder()).setNameFormat("DiscoveryClient-%d").setDaemon(true).build());
                // 创建心跳定时线程池
                this.heartbeatExecutor = new ThreadPoolExecutor(1, this.clientConfig.getHeartbeatExecutorThreadPoolSize(), 0L, TimeUnit.SECONDS, new SynchronousQueue(), (new ThreadFactoryBuilder()).setNameFormat("DiscoveryClient-HeartbeatExecutor-%d").setDaemon(true).build());
                this.cacheRefreshExecutor = new ThreadPoolExecutor(1, this.clientConfig.getCacheRefreshExecutorThreadPoolSize(), 0L, TimeUnit.SECONDS, new SynchronousQueue(), (new ThreadFactoryBuilder()).setNameFormat("DiscoveryClient-CacheRefreshExecutor-%d").setDaemon(true).build());
                this.eurekaTransport = new DiscoveryClient.EurekaTransport();
                this.scheduleServerEndpointTask(this.eurekaTransport, args);
                Object azToRegionMapper;
                if (this.clientConfig.shouldUseDnsForFetchingServiceUrls()) {
                    azToRegionMapper = new DNSBasedAzToRegionMapper(this.clientConfig);
                } else {
                    azToRegionMapper = new PropertyBasedAzToRegionMapper(this.clientConfig);
                }

                if (null != this.remoteRegionsToFetch.get()) {
                    ((AzToRegionMapper)azToRegionMapper).setRegionsToFetch(((String)this.remoteRegionsToFetch.get()).split(","));
                }

                this.instanceRegionChecker = new InstanceRegionChecker((AzToRegionMapper)azToRegionMapper, this.clientConfig.getRegion());
            } catch (Throwable var9) {
                throw new RuntimeException("Failed to initialize DiscoveryClient!", var9);
            }
            // fetchRegistry 从服务端拉取数据
            //shouldFetchRegistry()对应配置文件 eureka.client.fetch-registry 默认值为ture
            if (this.clientConfig.shouldFetchRegistry() && !this.fetchRegistry(false)) {
                //如果 eureka server 服务不可用，则采用的备用方案。 
                this.fetchRegistryFromBackup();
            }

            if (this.preRegistrationHandler != null) {
                this.preRegistrationHandler.beforeRegistration();
            }

            //shouldRegisterWithEureka() 对应配置属性eureka.client.registration.enabled
            if (this.clientConfig.shouldRegisterWithEureka() && this.clientConfig.shouldEnforceRegistrationAtInit()) {
                try {
                    //向注册中心注册
                    if (!this.register()) {
                        throw new IllegalStateException("Registration error at startup. Invalid server response.");
                    }
                } catch (Throwable var8) {
                    logger.error("Registration error at startup: {}", var8.getMessage());
                    throw new IllegalStateException(var8);
                }
            }

            this.initScheduledTasks();

            try {
                Monitors.registerObject(this);
            } catch (Throwable var7) {
                logger.warn("Cannot register timers", var7);
            }

            DiscoveryManager.getInstance().setDiscoveryClient(this);
            DiscoveryManager.getInstance().setEurekaClientConfig(config);
            this.initTimestampMs = System.currentTimeMillis();
            logger.info("Discovery Client initialized at timestamp {} with initial instances count: {}", this.initTimestampMs, this.getApplications().size());
        }
    }

```
## 拉取注册信息
```java
private boolean fetchRegistry(boolean forceFullRegistryFetch) {
        Stopwatch tracer = this.FETCH_REGISTRY_TIMER.start();

        label122: {
            boolean var4;
            try {
                Applications applications = this.getApplications();
                if (!this.clientConfig.shouldDisableDelta() && Strings.isNullOrEmpty(this.clientConfig.getRegistryRefreshSingleVipAddress()) && !forceFullRegistryFetch && applications != null && applications.getRegisteredApplications().size() != 0 && applications.getVersion() != -1L) {
                    this.getAndUpdateDelta(applications); //实际获取方法
                } else {
                    logger.info("Disable delta property : {}", this.clientConfig.shouldDisableDelta());
                    logger.info("Single vip registry refresh property : {}", this.clientConfig.getRegistryRefreshSingleVipAddress());
                    logger.info("Force full registry fetch : {}", forceFullRegistryFetch);
                    logger.info("Application is null : {}", applications == null);
                    logger.info("Registered Applications size is zero : {}", applications.getRegisteredApplications().size() == 0);
                    logger.info("Application version is -1: {}", applications.getVersion() == -1L);
                    this.getAndStoreFullRegistry();
                }

                applications.setAppsHashCode(applications.getReconcileHashCode());
                this.logTotalInstances();
                break label122;
            } catch (Throwable var8) {
                logger.error("DiscoveryClient_{} - was unable to refresh its cache! status = {}", new Object[]{this.appPathIdentifier, var8.getMessage(), var8});
                var4 = false;
            } finally {
                if (tracer != null) {
                    tracer.stop();
                }

            }

            return var4;
        }

        this.onCacheRefreshed();
        this.updateInstanceRemoteStatus();
        return true;
    }
```

## 服务注册  通过eurekaTransport 调用向 eureka server 进行服务注册
```java
    boolean register() throws Throwable {
        logger.info("DiscoveryClient_{}: registering service...", this.appPathIdentifier);

        EurekaHttpResponse httpResponse;
        try {
            httpResponse = this.eurekaTransport.registrationClient.register(this.instanceInfo);
        } catch (Exception var3) {
            logger.warn("DiscoveryClient_{} - registration failed {}", new Object[]{this.appPathIdentifier, var3.getMessage(), var3});
            throw var3;
        }

        if (logger.isInfoEnabled()) {
            logger.info("DiscoveryClient_{} - registration status: {}", this.appPathIdentifier, httpResponse.getStatusCode());
        }

        return httpResponse.getStatusCode() == Status.NO_CONTENT.getStatusCode();
    }
```

# 心跳机制
心跳机制的初始化工作也是在 DiscoveryClient 构造函数中完成。在DiscoveryClient构造函数的最后，有一个初始化调度任务的方法，在这个方法里就包括心跳的初始化。

创建线程池
```java
    his.heartbeatExecutor = new ThreadPoolExecutor(1, this.clientConfig.getHeartbeatExecutorThreadPoolSize(), 0L, TimeUnit.SECONDS, new SynchronousQueue(), (new ThreadFactoryBuilder()).setNameFormat("DiscoveryClient-HeartbeatExecutor-%d").setDaemon(true).build());
```

//TimedSupervisorTask 
TimedSupervisorTask 是 eureka 中自动调节间隔的周期性任务类。HeartbeatThread 是具体执行任何的线程，run方法中执行的就是 renew() 续期。
```java
    this.scheduler.schedule(new TimedSupervisorTask("heartbeat", this.scheduler, this.heartbeatExecutor, renewalIntervalInSecs, TimeUnit.SECONDS, expBackOffBound, new DiscoveryClient.HeartbeatThread()), (long)renewalIntervalInSecs, TimeUnit.SECONDS);

```
执行renew()
```java
    private class HeartbeatThread implements Runnable {
        private HeartbeatThread() {
        }

        public void run() {
            if (DiscoveryClient.this.renew()) {
                DiscoveryClient.this.lastSuccessfulHeartbeatTimestamp = System.currentTimeMillis();
            }

        }
    }
```
renew() 方法
```java
    boolean renew() {
        try {
            // 通过eurekaTransport 进行调用
            EurekaHttpResponse<InstanceInfo> httpResponse = this.eurekaTransport.registrationClient.sendHeartBeat(this.instanceInfo.getAppName(), this.instanceInfo.getId(), this.instanceInfo, (InstanceStatus)null);
            logger.debug("DiscoveryClient_{} - Heartbeat status: {}", this.appPathIdentifier, httpResponse.getStatusCode());
            // 404 未发现
            if (httpResponse.getStatusCode() == Status.NOT_FOUND.getStatusCode()) {
                this.REREGISTER_COUNTER.increment();
                logger.info("DiscoveryClient_{} - Re-registering apps/{}", this.appPathIdentifier, this.instanceInfo.getAppName());
                long timestamp = this.instanceInfo.setIsDirtyWithTime();
                boolean success = this.register(); // 重新注册
                if (success) {
                    this.instanceInfo.unsetIsDirty(timestamp);
                }
                return success;
            } else {
                return httpResponse.getStatusCode() == Status.OK.getStatusCode();
            }
        } catch (Throwable var5) {
            logger.error("DiscoveryClient_{} - was unable to send heartbeat!", this.appPathIdentifier, var5);
            return false;
        }
    }
```

# 总结
1. 注册中心启动后生成一系列REST接口,ApplicationResource,注册接口InstanceResource续约接口


2. 服务端启动后通过DiscoveryClient#register()向服务端注册

3. 服务端通过DiscoveryClient#getAndUpdateDelta()获取列表

4. 通过DiscoveryClient#renew()进行续约