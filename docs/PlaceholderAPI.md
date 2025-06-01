---
sidebar_label: 'PlaceholderAPI'
sidebar_position: 2
---

# PlaceholderAPI

PlaceholderAPI 是一个功能强大的 Minecraft 服务器插件，允许开发者通过占位符（placeholders）在聊天、计分板、命令等场景中显示动态信息。本文档基于 [PlaceholderAPI Wiki](https://wiki.placeholderapi.com/) 整理，详细介绍了如何在插件中使用现有占位符、创建自定义占位符扩展，以及其他高级功能和常见用例。以下内容为开发者提供清晰的 API 使用指南和示例代码，适用于 Bukkit/Spigot 插件开发。

## 1. 简介

PlaceholderAPI 是一个为 Minecraft 服务器设计的插件，允许插件通过统一的占位符系统共享和显示动态数据。它支持超过 240 个扩展，与多种流行插件（如 Vault、Essentials、LuckPerms）兼容，广泛应用于聊天格式、计分板、命令响应等场景。

### 1.1 什么是占位符？

占位符是一个字符串，表示动态数据，例如玩家的名字、健康值或服务器状态。例如，`%player_name%` 在解析时会被替换为玩家的实际名字。占位符通常以 `%identifier_parameter%` 格式出现，其中 `identifier` 是扩展的标识符，`parameter` 是具体的占位符参数。

### 1.2 PlaceholderAPI 的优势

- **标准化**：提供统一的接口，简化跨插件数据访问。
- **可扩展性**：支持开发者创建自定义占位符扩展。
- **兼容性**：支持多种插件和 Minecraft 版本（包括 1.21）。
- **社区支持**：拥有活跃的社区和扩展云（eCloud），提供丰富的预置占位符。

### 1.3 适用场景

- 在聊天消息中显示玩家信息（如等级、前缀）。
- 在计分板上展示实时数据（如在线人数、玩家统计）。
- 在命令响应中嵌入动态内容。
- 支持关系型占位符，用于比较两个玩家的数据（如朋友关系）。

## 2. 在插件中使用占位符

开发者可以通过 PlaceholderAPI 的 API 解析其他插件提供的占位符，用于动态显示数据。以下是使用占位符的步骤和方法。

### 2.1 设置依赖

要在插件中使用 PlaceholderAPI，需在构建工具和 `plugin.yml` 中配置依赖。

#### 2.1.1 Maven 配置
在 `pom.xml` 中添加以下依赖：

```xml
<dependency>
    <groupId>me.clip</groupId>
    <artifactId>placeholderapi</artifactId>
    <version>{version}</version>
    <scope>provided</scope>
</dependency>
```

#### 2.1.2 Gradle 配置
在 `build.gradle` 中添加：

```groovy
compileOnly 'me.clip:placeholderapi:{version}'
```

将 `{version}` 替换为最新版本，可在 [GitHub Releases](https://github.com/PlaceholderAPI/PlaceholderAPI/releases) 查看。

#### 2.1.3 配置 plugin.yml
在 `plugin.yml` 中添加 PlaceholderAPI 作为依赖：

**软依赖**（可选）：
```yaml
softdepend: [PlaceholderAPI]
```

**硬依赖**（必需）：
```yaml
depend: [PlaceholderAPI]
```

软依赖允许插件在 PlaceholderAPI 缺失时仍可加载，但需在代码中处理缺失情况。

### 2.2 检查 PlaceholderAPI 是否存在

在使用 API 方法之前，需检查 PlaceholderAPI 是否已安装，以避免空指针异常。

**示例**：
```java
if (Bukkit.getPluginManager().getPlugin("PlaceholderAPI") != null) {
    // 使用 PlaceholderAPI 方法
} else {
    getLogger().warning("PlaceholderAPI 未安装，相关功能将被禁用。");
}
```

### 2.3 解析占位符

使用 `PlaceholderAPI.setPlaceholders` 方法解析字符串中的占位符，支持在线玩家和离线玩家。

#### 2.3.1 针对在线玩家的占位符
**方法**：`String setPlaceholders(Player player, String text)`

**示例** - 自定义加入消息：
```java
import me.clip.placeholderapi.PlaceholderAPI;
import org.bukkit.event.player.PlayerJoinEvent;

public void onPlayerJoin(PlayerJoinEvent event) {
    String joinText = "%player_name% 加入了服务器！他们的等级是 %vault_rank%";
    if (Bukkit.getPluginManager().getPlugin("PlaceholderAPI") != null) {
        String parsedText = PlaceholderAPI.setPlaceholders(event.getPlayer(), joinText);
        getServer().broadcastMessage(parsedText);
    } else {
        getServer().broadcastMessage(event.getPlayer().getName() + " 加入了服务器！");
    }
}
```

#### 2.3.2 针对离线玩家的占位符
**方法**：`String setPlaceholders(OfflinePlayer player, String text)`

**示例**：
```java
import me.clip.placeholderapi.PlaceholderAPI;
import org.bukkit.OfflinePlayer;

public void displayPlayerInfo(OfflinePlayer offlinePlayer) {
    String text = "玩家 %player_name% 的余额是 %vault_eco_balance%";
    if (Bukkit.getPluginManager().getPlugin("PlaceholderAPI") != null) {
        String parsedText = PlaceholderAPI.setPlaceholders(offlinePlayer, text);
        getLogger().info(parsedText);
    }
}
```

### 2.4 批量解析占位符
**方法**：`List<String> setPlaceholders(Player player, List<String> text)`

用于解析包含占位符的字符串列表，适合处理多行文本（如计分板）。

**示例**：
```java
List<String> lines = Arrays.asList(
    "玩家: %player_name%",
    "等级: %vault_rank%",
    "余额: %vault_eco_balance%"
);
if (Bukkit.getPluginManager().getPlugin("PlaceholderAPI") != null) {
    List<String> parsedLines = PlaceholderAPI.setPlaceholders(player, lines);
    parsedLines.forEach(line -> player.sendMessage(line));
}
```

## 3. 创建自定义占位符（扩展）

开发者可以通过创建占位符扩展（PlaceholderExpansion）为其他插件提供自定义占位符。扩展可以是内部的（嵌入插件）或外部的（独立 Jar 文件）。

### 3.1 什么是占位符扩展？

占位符扩展是一个继承 `PlaceholderExpansion` 类的 Java 类，用于定义自定义占位符。占位符格式为 `%identifier_parameter%`，其中 `identifier` 是扩展的唯一标识符，`parameter` 是占位符的具体参数。

### 3.2 内部扩展与外部扩展

| 类型       | 描述                                   | 优点                                   | 缺点                                   | 注册方式                     |
|------------|----------------------------------------|----------------------------------------|----------------------------------------|------------------------------|
| 内部扩展   | 嵌入在插件内部，与插件一起加载         | 无需额外 Jar，易于访问插件数据         | 需手动注册，需设置 `persist()` 为 true | 在 `onEnable` 中手动注册     |
| 外部扩展   | 独立 Jar 文件，放入 `expansions` 文件夹 | 自动加载，支持 eCloud 分发             | 设置复杂，需检查依赖插件               | 自动加载，无需手动注册       |

### 3.3 创建扩展

需继承 `PlaceholderExpansion` 类并重写以下方法：
- `getIdentifier()`：返回扩展的唯一标识符（如 "myplugin"）。
- `getAuthor()`：返回作者名称。
- `getVersion()`：返回扩展版本。
- `onPlaceholderRequest(Player player, String identifier)`：处理占位符请求，返回值。

**示例** - 简单占位符扩展：
```java
import me.clip.placeholderapi.expansion.PlaceholderExpansion;
import org.bukkit.entity.Player;

public class MyExpansion extends PlaceholderExpansion {
    @Override
    public String getIdentifier() {
        return "myplugin";
    }

    @Override
    public String getAuthor() {
        return "YourName";
    }

    @Override
    public String getVersion() {
        return "1.0.0";
    }

    @Override
    public String onPlaceholderRequest(Player player, String identifier) {
        if (player == null) {
            return "";
        }
        if (identifier.equals("example")) {
            return "你好，" + player.getName();
        }
        return null;
    }
}
```

此扩展支持占位符 `%myplugin_example%`，返回 "你好，[玩家名字]"。

### 3.4 注册扩展

#### 3.4.1 内部扩展
在插件的 `onEnable` 方法中注册：

```java
public void onEnable() {
    if (Bukkit.getPluginManager().getPlugin("PlaceholderAPI") != null) {
        new MyExpansion().register();
    }
}
```

#### 3.4.2 外部扩展
将扩展打包为 Jar 文件，放入 `plugins/PlaceholderAPI/expansions/` 文件夹，PlaceholderAPI 会自动加载。

#### 3.4.3 设置持久化
对于内部扩展，需重写 `persist()` 方法以确保扩展在服务器重载时不被卸载：

```java
@Override
public boolean persist() {
    return true;
}
```

### 3.5 关系型占位符

关系型占位符用于比较两个玩家的数据，需实现 `Relational` 接口。占位符以 `rel_` 开头。

**示例**：
```java
import me.clip.placeholderapi.expansion.PlaceholderExpansion;
import me.clip.placeholderapi.expansion.Relational;
import org.bukkit.entity.Player;

public class MyRelationalExpansion extends PlaceholderExpansion implements Relational {
    @Override
    public String getIdentifier() {
        return "myrelational";
    }

    @Override
    public String getAuthor() {
        return "YourName";
    }

    @Override
    public String getVersion() {
        return "1.0.0";
    }

    @Override
    public String onPlaceholderRequest(Player one, Player two, String identifier) {
        if (one == null || two == null) {
            return "";
        }
        if (identifier.equals("are_friends")) {
            // 假设存在检查朋友关系的逻辑
            return areFriends(one, two) ? "朋友" : "非朋友";
        }
        return null;
    }
}
```

占位符 `%rel_myrelational_are_friends%` 可用于计分板等支持关系型占位符的场景。

## 4. 高级主题

### 4.1 占位符格式

占位符通常遵循 `%identifier_parameter%` 格式。例如：
- `%player_name%`：`player` 是标识符，`name` 是参数。
- `%vault_eco_balance%`：`vault` 是标识符，`eco_balance` 是参数。

关系型占位符以 `%rel_identifier_parameter%` 格式出现。

### 4.2 动态占位符

动态占位符允许在参数中传递额外信息，需在 `onPlaceholderRequest` 方法中解析 `identifier`。

**示例**：
```java
@Override
public String onPlaceholderRequest(Player player, String identifier) {
    if (player == null) {
        return "";
    }
    if (identifier.startsWith("stat_")) {
        String statType = identifier.substring(5); // 提取 stat_ 后的部分
        return getPlayerStat(player, statType);
    }
    return null;
}
```

占位符如 `%myplugin_stat_kills%` 可返回玩家的击杀数。

### 4.3 eCloud 扩展

外部扩展可上传至 PlaceholderAPI 的扩展云（eCloud），通过 `/papi ecloud download <expansion>` 命令下载。需确保扩展正确配置 `getRequiredPlugin()` 和 `canRegister()` 方法。

**示例**：
```java
@Override
public String getRequiredPlugin() {
    return "MyPlugin";
}

@Override
public boolean canRegister() {
    return Bukkit.getPluginManager().getPlugin(getRequiredPlugin()) != null;
}
```

## 5. 常见用例

| 用例             | 描述                                   | 示例占位符                     |
|------------------|----------------------------------------|--------------------------------|
| 自定义加入消息   | 在玩家加入时显示动态信息               | `%player_name%`, `%vault_rank%` |
| 动态计分板       | 在计分板上显示实时数据                 | `%player_health%`, `%server_online%` |
| 聊天格式化       | 在聊天中添加前缀、后缀或统计信息       | `%vault_prefix%`, `%myplugin_stat_kills%` |
| 命令响应         | 在命令输出中嵌入动态数据               | `%player_uuid%`, `%vault_eco_balance%` |

## 6. 故障排除

- **占位符无效**：使用 `/papi parse <player> <placeholder>` 测试占位符，确保相关扩展已安装并启用。
- **依赖问题**：检查 `plugin.yml` 和构建工具中的依赖配置，确保版本兼容。
- **性能问题**：频繁解析占位符可能影响性能，建议缓存结果或限制解析频率。
- **关系型占位符不支持**：确保使用场景（如计分板）支持 `rel_` 占位符。

更多故障排除信息，请参阅 [PlaceholderAPI Wiki](https://wiki.placeholderapi.com/)。

## 7. 资源与支持

- **官方文档**：查看 [PlaceholderAPI Wiki](https://wiki.placeholderapi.com/) 获取最新信息。
- **扩展云**：通过 `/papi ecloud` 命令浏览和下载扩展。
- **社区支持**：在 GitHub Issues 或 PlaceholderAPI 的 Discord 社区寻求帮助。
