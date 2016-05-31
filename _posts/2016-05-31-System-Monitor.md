---
layout: post_layout
title: 简单性能监控
time: 2016年05月31日 星期二
location: 北京
pulished: true
tag: java
excerpt_separator: "``"
---

分享一个最近写的用于收集系统以及jvm性能指标数据的JAVA代码

有时候我们有需要监控节点的磁盘、内存、cpu以及JVM的内存使用情况等性能指标的需求，不想用复杂的框架或APM工具，可以试试下面这个：

``

#### SystemMonitor.java ####

监控主类，实例化后默认添加四个性能抓取器：物理内存、JVM内存、Cpu、磁盘，当然你也可以自己选择所需要的性能抓取器，当你调用getSystemInfos方法后，会抓取Monitor中包含的抓取器的性能数据

```java
    import java.util.ArrayList;
    import java.util.List;
    import java.util.stream.Collectors;
    
    import io.vertx.core.json.JsonObject;
    
    /**
     *
     * <code>SystemMonitor<code>
     * <strong></strong>
     * <p>说明：系统性能监控
     * <li></li>
     * </p>
     * 
     */
    
    public class SystemMonitor {
      private List<SystemInfoFetcher> fetcherList = new ArrayList<SystemInfoFetcher>();
    
      /**
       * 默认添加物理内存、jvm内存、cpu、存储信息抓取器
       * 
       */
      public SystemMonitor() {
    	fetcherList.add(new PhysicalMemoryFetcher());
    	fetcherList.add(new JvmMemoryFetcher());
    	fetcherList.add(new CpuFetcher());
    	fetcherList.add(new StorageFetcher());
      }
    
      /**
       * 自定义想要抓取的系统性能信息
       *
       * @param list 的构造子
       */
      public SystemMonitor(List<String> list) {
    	for (String fetcherType : list) {
      		fetcherList.add(SystemInfoFetcherFactory.create(fetcherType));
    	}
      }
    
      /**
       * 
       * <p>
       * 说明：添加一个性能抓取器
       * <li></li>
       * </p>
       * @param fetcherType
       * @return
       */
      public boolean addFetcher(String fetcherType) {
    	if (containsFetcher(fetcherType))
      		return false;
    	else {
      		fetcherList.add(SystemInfoFetcherFactory.create(fetcherType));
      		return true;
    	}
      }
    
      /**
       * 
       * <p>
       * 说明：删除一个性能抓取器
       * <li></li>
       * </p>
       * 
       * @param fetcherType
       */
      public void removeFetcher(String fetcherType) {
    	fetcherList =
    	fetcherList.stream().filter(fetcher -> !fetcher.infoType().equals(fetcherType))
    	.collect(Collectors.toList());
      }
    
      /**
       * 
       * <p>
       * 说明：删除所有性能抓取器
       * <li></li>
       * </p>
       * 
       * @date 2016年5月11日 下午4:25:20
       */
      public void removeAllFetchers() {
    	fetcherList.clear();
      }
    
      /**
       * 
       * <p>
       * 说明：添加一组性能抓取器
       * <li></li>
       * </p>
       * 
       * @param fetcherList
       */
      public void addFetcherList(List<String> fetcherList) {
    	for (String fetcherType : fetcherList) {
      		addFetcher(fetcherType);
    	}
      }
    
      /**
       * 
       * <p>
       * 说明：判断特定类型的性能抓取器是否存在
       * <li></li>
       * </p>
       * 
       * @param type
       * @return
       */
      public boolean containsFetcher(String type) {
    	long count = fetcherList.stream().filter(fetcher -> fetcher.infoType().equals(type)).count();
    	if (count > 0)
      		return true;
    	else
      		return false;
      }
    
      /**
       * 
       * <p>
       * 说明：获取各性能抓取器获取的性能信息
       * <li></li>
       * </p>
       * 
       * @return
       */
      public JsonObject getSystemInfos() {
    	JsonObject sysInfos = new JsonObject();
    
    	for (SystemInfoFetcher fetcher : fetcherList) {
      		sysInfos.mergeIn(fetcher.fetch());
    	}
    	return sysInfos;
      }
    
    }
```

#### SystemInfoFetcher ####
性能抓取器接口，每一个性能抓取器都需要实现该接口，我们默认实现了四类，如上所说有：CPU、磁盘、物理内存、JVM内存，如果有自己还想抓取的性能数据，可以自己实现该接口

```java
    import io.vertx.core.json.JsonObject;
    
    /**
     *
     * <code>SystemInfoFetcher<code>
     * <strong></strong>
     * <p>说明：系统信息抓取接口
     * <li></li>
     * </p>
     * 
     */
    public interface SystemInfoFetcher {
      
	  /**
       * 
       * <p>
       * 说明：对相关信息进行抓取
       * <li></li>
       * </p>
       */
      public JsonObject fetch();
    
      /**
       * 
       * <p>
       * 说明：返回抓取信息的类型
       * <li></li>
       * </p>
       */
      public String infoType();
    }
```

#### CpuFetcher ####
抓取CPU信息

```java
    import java.lang.management.ManagementFactory;
    import com.sun.management.OperatingSystemMXBean;
    
    import io.vertx.core.json.JsonObject;
    
    /**
     *
     * <code>CpuInfoFetcher<code>
     * <strong></strong>
     * <p>说明：Cpu相关信息抓取
     * <li></li>
     * </p>
     * 
     */
    
    @SuppressWarnings("restriction")
    public class CpuFetcher implements SystemInfoFetcher {
    
      private JsonObject infos = new JsonObject();
    
      @Override
      public JsonObject fetch() {
    	OperatingSystemMXBean osmxb =
    	(OperatingSystemMXBean) ManagementFactory.getOperatingSystemMXBean();
    
    	double jvmCpuLoad = osmxb.getProcessCpuLoad();
    	infos.put(SystemInfoSchema.JVM_CPU_LOAD, jvmCpuLoad);
    
    	double systemCpuLoad = osmxb.getSystemCpuLoad();
    	infos.put(SystemInfoSchema.SYSTEM_CPU_LOAD, systemCpuLoad);
    
    	return infos;
      }
    
      @Override
      public String infoType() {
    	return SystemInfoSchema.CPU;
      }
    
    }
```

#### JvmMemoryFetcher ####

```java
    import io.vertx.core.json.JsonObject;
    
    /**
     *
     * <code>JvmMemoryFetcher<code>
     * <strong></strong>
     * <p>说明：Jvm内存信息抓取
     * <li></li>
     * </p>
     */
    
    public class JvmMemoryFetcher implements SystemInfoFetcher {
    
      private JsonObject infos = new JsonObject();
      private long Mb = 1024 * 1024;
    
      @Override
      public JsonObject fetch() {
    	// 可使用内存
    	long totalMemory = Runtime.getRuntime().totalMemory() / Mb;
    	infos.put(SystemInfoSchema.JVM_MEMORY_TOTAL, totalMemory);
    
    	// 剩余内存
    	long freeMemory = Runtime.getRuntime().freeMemory() / Mb;
    	infos.put(SystemInfoSchema.JVM_MEMORY_FREE, freeMemory);
    
    	// 最大可使用内存
    	long maxMemory = Runtime.getRuntime().maxMemory() / Mb;
    	infos.put(SystemInfoSchema.JVM_MEMORY_MAX, maxMemory);
    
    	// 已使用内存
    	long usedMemory = totalMemory - freeMemory;
    	infos.put(SystemInfoSchema.JVM_MEMORY_USED, usedMemory);
    
    	return infos;
      }
    
      @Override
      public String infoType() {
    	return SystemInfoSchema.JVM_MEMORY;
      }
    
    }
```

#### PhysicalMemoryFetcher ####

```java
    import java.lang.management.ManagementFactory;
    import com.sun.management.OperatingSystemMXBean;
    import io.vertx.core.json.JsonObject;
    
    /**
     *
     * <code>PhysicalMemoryFetcher<code>
     * <strong></strong>
     * <p>说明：物理内存信息抓取
     * <li></li>
     * </p>
     * 
     */
    
    @SuppressWarnings("restriction")
    public class PhysicalMemoryFetcher implements SystemInfoFetcher {
    
      private JsonObject infos = new JsonObject();
      private long Mb = 1024 * 1024;
    
      @Override
      public JsonObject fetch() {
    	OperatingSystemMXBean osmxb =
    	(OperatingSystemMXBean) ManagementFactory.getOperatingSystemMXBean();
    
    	// 总的物理内存
    	long totalMemorySize = osmxb.getTotalPhysicalMemorySize() / Mb;
    	infos.put(SystemInfoSchema.PHYSICAL_MEMORY_TOTAL, totalMemorySize);
    
    	// 剩余的物理内存
    	long freeMemorySize = osmxb.getFreePhysicalMemorySize() / Mb;
    	infos.put(SystemInfoSchema.PHYSICAL_MEMORY_FREE, freeMemorySize);
    
    	// 使用的物理内存
    	long usedMemorySize = totalMemorySize - freeMemorySize;
    	infos.put(SystemInfoSchema.PHYSICAL_MEMORY_USED, usedMemorySize);
    
    	return infos;
      }
    
      @Override
      public String infoType() {
    	return SystemInfoSchema.PHYSICAL_MEMORY;
      }
    
    }
```

#### StorageFetcher #### 

```java
    import java.io.File;
    import io.vertx.core.json.JsonObject;
    
    /**
     *
     * <code>StorageFetcher<code>
     * <strong></strong>
     * <p>说明：磁盘信息获取
     * <li></li>
     * </p>
     * 
     */
    
    public class StorageFetcher implements SystemInfoFetcher {
    
      private JsonObject infos = new JsonObject();
      private double Gb = 1024 * 1024 * 1024;
    
      @Override
      public JsonObject fetch() {
    	// 硬盘总大小
    	int totalStorage = 0;
		// 已用硬盘大小
    	int usedStorage = 0;
    	// 剩余硬盘大小
    	int freeStorage = 0;
    
    	File[] roots = File.listRoots();
    
    	for (File _file : roots) {
      		// 获取硬盘总大小
      		totalStorage += _file.getTotalSpace() / Gb;
      		// 获取剩余硬盘大小
      		freeStorage += _file.getFreeSpace() / Gb;
    	}
    	usedStorage = totalStorage - freeStorage;
    
    	infos.put(SystemInfoSchema.STORAGE_TOTAL, totalStorage);
    	infos.put(SystemInfoSchema.STORAGE_FREE, freeStorage);
    	infos.put(SystemInfoSchema.STORAGE_USED, usedStorage);
    
    	return infos;
      }
    
      @Override
      public String infoType() {
    	return SystemInfoSchema.STORAGE;
      }
    
    }
```
    
#### SystemInfoFetcherFactory ####
性能抓取器工厂类，根据性能名称实例化抓取器

```java
    /**
     *
     * <code>SystemInfoFetcherFactory<code>
     * <strong></strong>
     * <p>说明：系统性能抓取器工厂
     * <li></li>
     * </p>
     * 
     */
    
    public class SystemInfoFetcherFactory {
    
      public static SystemInfoFetcher create(String type) {
    	switch (type) {
      		case SystemInfoSchema.PHYSICAL_MEMORY:
				return new PhysicalMemoryFetcher();
      		case SystemInfoSchema.JVM_MEMORY:
    			return new JvmMemoryFetcher();
      		case SystemInfoSchema.CPU:
    			return new CpuFetcher();
      		case SystemInfoSchema.STORAGE:
    			return new StorageFetcher();
      		default:
    			throw new IllegalArgumentException("system info fetcher type : " + type + " is not defined, please check SystemInfoSchema for defined fetcher type");
    	}
      }
    }
```    

#### SystemInfoSchema ####

```java
    /**
     *
     * <code>SystemInfoSchema<code>
     * <strong></strong>
     * <p>说明：系统性能数据常量
     * <li></li>
     * </p>
     * 
     */
    
    public class SystemInfoSchema {
      // System Info Fetcher Type
      public static final String PHYSICAL_MEMORY = "physical_memory";
      public static final String JVM_MEMORY = "jvm_memory";
      public static final String STORAGE = "storage";
      public static final String CPU = "cpu";
    
      // Physical Memory
      public static final String PHYSICAL_MEMORY_TOTAL = "physical_memory_total";
      public static final String PHYSICAL_MEMORY_FREE = "physical_memory_free";
      public static final String PHYSICAL_MEMORY_USED = "physical_memory_used";
    
      // Jvm Memory
      public static final String JVM_MEMORY_TOTAL = "jvm_memory_total";
      public static final String JVM_MEMORY_FREE = "jvm_memory_free";
      public static final String JVM_MEMORY_MAX = "jvm_memory_max";
      public static final String JVM_MEMORY_USED = "jvm_memory_used";
    
      // Storage
      public static final String STORAGE_TOTAL = "storage_total";
      public static final String STORAGE_FREE = "storage_free";
      public static final String STORAGE_USED = "storage_used";
    
      // Cpu
      public static final String JVM_CPU_LOAD = "jvm_cpu_load";
      public static final String SYSTEM_CPU_LOAD = "system_cpu_load";
    
    }
```

如下两行简单代码就可以获取性能数据：

```java
SystemMonitor sysMonitor = new SystemMonitor();
JsonObject sysInfos = sysMonitor.getSystemInfos();
```

	
