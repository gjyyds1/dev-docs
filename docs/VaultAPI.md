---
sidebar_label: 'VaultAPI'
sidebar_position: 1
---

# VaultAPI 用法与用途文档

整理自 [VaultAPI 1.7 Javadoc](https://milkbowl.github.io/VaultAPI/)
文档创建时间: 2025年5月28日
语言: 中文

---

## 1. 概述

Vault 是一个为 Bukkit/Spigot 插件提供的统一接口，主要用于权限（Permission）、聊天（Chat）和经济（Economy）系统的集成。它通过抽象层屏蔽了底层实现的差异，使插件开发者无需针对每个具体的权限或经济插件进行适配。

**主要用途**：

* 统一访问和操作权限、聊天和经济系统。
* 简化插件与多种权限/经济插件的兼容性处理。
* 提供标准化的 API 接口，减少重复开发工作。

---

## 2. 设置依赖

### 2.1 plugin.yml 配置

在 `plugin.yml` 文件中添加对 Vault 的依赖：

```yaml
softdepend: [Vault] # 或者depend: [Vault]
```

### 2.2 Maven 依赖配置

在 `pom.xml` 中添加 VaultAPI 的依赖：

```xml
<repository>
    <id>jitpack.io</id>
    <url>https://jitpack.io</url>
</repository>

<dependency>
    <groupId>com.github.MilkBowl</groupId>
    <artifactId>VaultAPI</artifactId>
    <version>1.7</version>
    <scope>provided</scope>
</dependency>
```

### 2.3 Gradle 依赖配置

在 `build.gradle.kts` 中添加 VaultAPI 的依赖：

```groovy
repositories {
    maven { url 'https://jitpack.io' }
}
dependencies {
    compileOnly "com.github.MilkBowl:VaultAPI:1.7"
}
```

---

## 3. 服务初始化

在插件的 `onEnable` 方法中，初始化 Vault 提供的服务：

```java
import net.milkbowl.vault.economy.Economy;
import net.milkbowl.vault.permission.Permission;
import net.milkbowl.vault.chat.Chat;
import org.bukkit.plugin.RegisteredServiceProvider;
import org.bukkit.plugin.JavaPlugin;

public class MyPlugin extends JavaPlugin {
    private Economy economy;
    private Permission permission;
    private Chat chat;

    @Override
    public void onEnable() {
        if (!setupEconomy()) {
            getLogger().severe("未找到经济服务，插件将被禁用。");
            getServer().getPluginManager().disablePlugin(this);
            return;
        }
        setupPermission();
        setupChat();
    }

    private boolean setupEconomy() {
        RegisteredServiceProvider<Economy> rsp = getServer().getServicesManager().getRegistration(Economy.class);
        if (rsp == null) return false;
        economy = rsp.getProvider();
        return economy != null;
    }

    private boolean setupPermission() {
        RegisteredServiceProvider<Permission> rsp = getServer().getServicesManager().getRegistration(Permission.class);
        if (rsp == null) return false;
        permission = rsp.getProvider();
        return permission != null;
    }

    private boolean setupChat() {
        RegisteredServiceProvider<Chat> rsp = getServer().getServicesManager().getRegistration(Chat.class);
        if (rsp == null) return false;
        chat = rsp.getProvider();
        return chat != null;
    }
}
```

---

## 4. 核心服务与用法

### 4.1 Economy（经济服务）

**用途**：管理玩家的经济账户，包括余额查询、存取款等操作。([milkbowl.github.io][1])

**主要方法**：

* `hasAccount(OfflinePlayer player)`：检查玩家是否有账户。
* `getBalance(OfflinePlayer player)`：获取玩家余额。
* `has(OfflinePlayer player, double amount)`：检查玩家是否有足够的余额。
* `withdrawPlayer(OfflinePlayer player, double amount)`：从玩家账户扣款。
* `depositPlayer(OfflinePlayer player, double amount)`：向玩家账户存款。

**示例**：

```java
if (economy != null && economy.has(player, 50.0)) {
    EconomyResponse response = economy.withdrawPlayer(player, 50.0);
    if (response.transactionSuccess()) {
        player.sendMessage("成功支付 50 单位货币。");
    } else {
        player.sendMessage("支付失败：" + response.errorMessage);
    }
} else {
    player.sendMessage("余额不足或经济服务不可用。");
}
```

---

### 4.2 Permission（权限服务）

**用途**：管理玩家和组的权限，包括添加、移除和检查权限节点。

**主要方法**：

* `has(Player player, String permission)`：检查玩家是否拥有指定权限。
* `playerAdd(String world, OfflinePlayer player, String permission)`：为玩家添加权限。
* `playerRemove(String world, OfflinePlayer player, String permission)`：移除玩家的权限。
* `groupAdd(String world, String group, String permission)`：为组添加权限。
* `groupRemove(String world, String group, String permission)`：移除组的权限。

**示例**：

```java
if (permission != null && permission.has(player, "myplugin.use")) {
    player.sendMessage("你有使用此插件的权限。");
} else {
    player.sendMessage("你没有使用此插件的权限。");
}
```

---

### 4.3 Chat（聊天服务）

**用途**：管理玩家和组的聊天前缀、后缀及其他元数据。

**主要方法**：

* `getPlayerPrefix(String world, OfflinePlayer player)`：获取玩家的前缀。
* `setPlayerPrefix(String world, OfflinePlayer player, String prefix)`：设置玩家的前缀。
* `getPlayerSuffix(String world, OfflinePlayer player)`：获取玩家的后缀。
* `setPlayerSuffix(String world, OfflinePlayer player, String suffix)`：设置玩家的后缀。
* `getGroupPrefix(String world, String group)`：获取组的前缀。
* `setGroupPrefix(String world, String group, String prefix)`：设置组的前缀。

**示例**：

```java
if (chat != null) {
    String prefix = chat.getPlayerPrefix(player.getWorld().getName(), player);
    player.sendMessage("你的前缀是：" + prefix);
    chat.setPlayerPrefix(player.getWorld().getName(), player, "[VIP]");
    player.sendMessage("已将你的前缀设置为 [VIP]。");
}
```

---

## 5. 注意事项

* **服务检查**：在使用任何 Vault 服务前，务必检查对应的服务提供者是否为 null，以避免空指针异常。
* **依赖声明**：在 `plugin.yml` 中使用 `softdepend` 声明对 Vault 的依赖，以确保插件在 Vault 存在时正确加载。
* **多世界支持**：某些权限和聊天服务支持多世界功能，使用相关方法时需指定世界名称。
* **兼容性**：Vault 仅提供统一接口，具体功能取决于底层实现插件的支持情况。

---

## 6. 资源链接

* [**VaultAPI Javadoc**](https://milkbowl.github.io/VaultAPI/)
* [**Vault GitHub 仓库**](https://github.com/MilkBowl/Vault)
* [**VaultAPI GitHub 仓库**](https://github.com/MilkBowl/VaultAPI)
* [**SpigotMC 插件页面**](https://www.spigotmc.org/resources/vault.34315/)
* [**Bukkit 插件页面**](https://dev.bukkit.org/projects/vault)

---

## 7. 总结

Vault 提供了一个统一的接口，使插件开发者能够方便地与多种权限、聊天和经济插件进行集成。通过使用 Vault，开发者无需针对每个具体的实现进行适配，从而提高了插件的兼容性和开发效率。在使用 Vault 时，务必注意服务的初始化和空值检查，以确保插件的稳定运行。
