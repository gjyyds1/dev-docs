# ProtocolLib API 用法与用途文档

整理自 ProtocolLib 5.4.0-SNAPSHOT Javadoc (<https://ci.dmulloy2.net/job/ProtocolLib/javadoc/>)  
文档创建时间: 2025年5月25日  
语言: 中文  

---

## 1. 概述
ProtocolLib 是一个强大的 Minecraft Bukkit/Spigot 插件库，用于读写和修改 Minecraft 的网络数据包。它通过事件驱动的 API 屏蔽了底层的 NMS（Net Minecraft Server）复杂性，提供简单的索引读写系统，减少对 CraftBukkit 的直接依赖。其主要目标是提供跨版本兼容性，通过反射机制动态解析字段、方法和类，适应 Minecraft 的不同版本。

**主要用途**：  
- 拦截、修改或取消客户端和服务器之间的数据包。  
- 实现自定义数据包功能，如伪造爆炸效果、修改聊天消息等。  
- 支持插件开发者实现无法通过标准 Bukkit API 完成的功能。

---

## 2. 核心类与用法

### 2.1 ProtocolManager
**用途**：ProtocolLib 的核心管理类，用于管理数据包监听器、发送数据包和访问协议相关功能。  

**获取方式**：  
```java
private ProtocolManager protocolManager;

@Override
public void onEnable() {
    protocolManager = ProtocolLibrary.getProtocolManager();
}
```

**主要方法**：  
- `addPacketListener(PacketListener listener)`：注册数据包监听器。  
- `sendServerPacket(Player player, PacketContainer packet)`：向指定玩家发送服务器数据包。  
- `createPacket(PacketType type)`：创建指定类型的数据包容器。  
- `removePacketListener(PacketListener listener)`：移除数据包监听器。  

**用法示例 - 禁用服务器发出的音效**：  
```java
protocolManager.addPacketListener(new PacketAdapter(this, ListenerPriority.NORMAL, PacketType.Play.Server.NAMED_SOUND_EFFECT) {
    @Override
    public void onPacketSending(PacketEvent event) {
        event.setCancelled(true); // 取消音效数据包
    }
});
```

**用途**：  
- 监听特定数据包并进行修改或取消。  
- 实现聊天审查、数据包过滤等功能。

---

### 2.2 PacketContainer
**用途**：数据包的包装类，用于读写数据包的内容，简化对复杂数据结构的访问。  

**主要方法**：  
- `getIntegers()`, `getStrings()`, `getDoubles()` 等：获取对应类型的数据修改器。  
- `write(int index, T value)`：写入指定索引的数据。  
- `read(int index)`：读取指定索引的数据。  

**用法示例 - 创建并发送伪造爆炸数据包**：  
```java
PacketContainer fakeExplosion = new PacketContainer(PacketType.Play.Server.EXPLOSION);
fakeExplosion.getDoubles()
    .write(0, player.getLocation().getX())
    .write(1, player.getLocation().getY())
    .write(2, player.getLocation().getZ());
fakeExplosion.getFloat().write(0, 3.0F);
fakeExplosion.getBlockPositionCollectionModifier().write(0, new ArrayList<>());
fakeExplosion.getVectors().write(0, player.getVelocity().add(new Vector(1, 1, 1)));
try {
    protocolManager.sendServerPacket(player, fakeExplosion);
} catch (InvocationTargetException e) {
    throw new RuntimeException("无法发送数据包", e);
}
```

**用途**：  
- 简化数据包字段的读写操作。  
- 创建或修改数据包以实现自定义效果（如伪造爆炸、修改实体位置等）。

---

### 2.3 PacketType
**用途**：枚举定义了所有支持的 Minecraft 数据包类型，按协议状态分类。  

**主要字段**：  
- `PacketType.Play.Server.*`：服务器发送到客户端的数据包（如 `NAMED_SOUND_EFFECT`, `EXPLOSION`）。  
- `PacketType.Play.Client.*`：客户端发送到服务器的数据包（如 `CHAT`, `POSITION`）。  
- `PacketType.Configuration.*`：配置阶段的数据包。  

**用途**：  
- 指定监听或创建的数据包类型。  
- 避免直接使用数据包 ID，确保跨版本兼容性。

---

### 2.4 WrappedChatComponent
**用途**：处理 JSON 格式的聊天组件。  

**主要方法**：  
- `fromChatMessage(String message)`：从字符串创建聊天组件。  
- `getJson()`：获取聊天组件的 JSON 表示。  
- `getHandle()`：获取底层 NMS 对象。  

**用法示例 - 修改聊天消息**：  
```java
PacketContainer packet = event.getPacket();
WrappedChatComponent chat = packet.getChatComponents().read(0);
String json = chat.getJson();
if (json.contains("hello")) {
    chat = WrappedChatComponent.fromText("你好！");
    packet.getChatComponents().write(0, chat);
}
```

**用途**：  
- 修改或创建复杂的聊天消息。  
- 支持 JSON 格式的聊天组件处理。

---

### 2.5 WrappedDataWatcher
**用途**：处理实体的数据观察者（DataWatcher），管理元数据。  

**主要方法**：  
- `setObject(int index, Object value)`：设置指定索引的元数据。  
- `getObject(int index)`：获取指定索引的元数据。  

**用法示例 - 修改实体显示名称**：  
```java
WrappedDataWatcher watcher = new WrappedDataWatcher();
watcher.setObject(2, WrappedDataWatcher.WrappedDataValue.fromString("CustomName"));
```

**用途**：  
- 修改实体属性（如隐身、燃烧效果等）。  
- 用于伪造实体或修改实体状态。

---

## 3. 高级功能

### 3.1 异步数据包处理
**相关类**：`com.comphenix.protocol.async.AsyncListenerHandler`  
**用途**：异步执行数据包操作，避免阻塞主线程。  

**用法示例**：  
```java
AsyncListenerHandler asyncHandler = protocolManager.getAsynchronousManager().registerAsyncHandler(
    new PacketAdapter(this, ListenerPriority.NORMAL, PacketType.Play.Server.NAMED_SOUND_EFFECT) {
        @Override
        public void onPacketSending(PacketEvent event) {
            // 异步处理逻辑
        }
    }
);
asyncHandler.start();
```

**用途**：  
- 提高性能，避免在主线程执行复杂逻辑。  
- 适合需要大量计算或延迟处理的操作。

---

### 3.2 网络注入
**相关类**：`com.comphenix.protocol.injector.netty.NettyChannelInjector`  
**用途**：直接注入网络通道拦截原始数据包。  
**注意**：通常无需直接使用，ProtocolManager 已封装相关功能。  

**用途**：  
- 高级开发者可用于自定义网络处理逻辑。  
- 需谨慎使用，可能影响性能或稳定性。

---

### 3.3 模糊匹配
**相关类**：`com.comphenix.protocol.reflect.fuzzy.FuzzyFieldContract`, `FuzzyMethodContract`  
**用途**：通过模糊匹配反射字段或方法，增强跨版本兼容性。  

**用法示例**：  
```java
FuzzyFieldContract contract = FuzzyFieldContract.newBuilder()
    .typeExact(String.class)
    .build();
```

**用途**：  
- 动态查找 NMS 中的字段或方法。  
- 确保代码在不同 Minecraft 版本中保持兼容。

---

## 4. 常见用途场景
- **聊天消息审查**：监听 `PacketType.Play.Client.CHAT`，检查并修改聊天内容。  
- **伪造效果**：使用 `PacketType.Play.Server.EXPLOSION` 创建视觉爆炸效果。  
- **实体修改**：通过 `WrappedDataWatcher` 修改元数据（如隐身或名称）。  
- **性能优化**：使用异步监听器处理高负载操作。  
- **自定义协议**：通过 `NettyChannelInjector` 实现高级协议修改。

---

## 5. 注意事项
- **版本兼容性**：需使用最新开发版支持新 Minecraft 版本（如 1.21.3）。  
- **异步处理**：优先使用 `AsyncListenerHandler` 避免阻塞主线程。  
- **调试**：使用 `/packetlog` 命令记录数据包日志。  
- **依赖管理**：在 `plugin.yml` 中添加 ProtocolLib 作为依赖，并在 `pom.xml` 中配置仓库：  
```yaml
depends: [ProtocolLib]
```
```xml
<repositories>
    <repository>
        <id>dmulloy2-repo</id>
        <url>https://repo.dmulloy2.net/repository/public/</url>
    </repository>
</repositories>
<dependencies>
    <dependency>
        <groupId>com.comphenix.protocol</groupId>
        <artifactId>ProtocolLib</artifactId>
        <version>5.4.0</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

---

## 6. 资源链接
- **Javadoc**: <https://ci.dmulloy2.net/job/ProtocolLib/javadoc/>  
- **GitHub**: <https://github.com/dmulloy2/ProtocolLib>  
- **问题追踪**: <https://github.com/dmulloy2/ProtocolLib/issues>  
- **赞助支持**: <https://github.com/sponsors/dmulloy2>  

---

## 7. 总结
ProtocolLib 提供了强大的数据包处理能力，简化了 Minecraft 插件开发中的网络操作。通过 `ProtocolManager`、`PacketAdapter` 和 `PacketContainer` 等核心类，开发者可以轻松实现数据包监听、修改和发送。高级功能如异步处理和模糊匹配进一步增强了其灵活性和兼容性。建议结合 Javadoc 和 GitHub 示例深入学习。
