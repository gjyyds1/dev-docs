---
sidebar_label: 'VaultAPI'
sidebar_position: 1
---

# VaultAPI 用法与用途文档

整理自 [VaultAPI 1.7 Javadoc](https://milkbowl.github.io/VaultAPI/)  
文档创建时间: 2025年5月28日  
语言: 中文  

---

# 概述
Vault 是一个用于 Bukkit/Spigot 插件的权限、聊天和经济 API，它提供了一种标准化的方式，让插件能够与各种经济、权限和聊天系统交互，而无需了解每种实现的细节。以下是 VaultAPI 的详细用法和示例，涵盖插件如何接入 VaultAPI 以及其主要 API 的使用方法。

## 关键要点
- **功能**：VaultAPI 允许插件与经济、权限和聊天系统交互，支持多种实现（如 Essentials、PermissionsEx）。
- **接入方式**：通过在 `plugin.yml` 中添加软依赖或硬依赖并在代码中检查服务提供者来接入 Vault。
- **主要服务**：包括经济（Economy）、权限（Permission）和聊天（Chat）服务。
- **注意事项**：需检查 Vault 和服务提供者是否存在，以避免空指针异常。
- **兼容性**：Vault 提供跨插件兼容性，但具体功能依赖于底层实现（如是否支持多世界经济）。

## 设置依赖
在插件的 `plugin.yml` 文件中，将 Vault 添加为软依赖，以确保插件在 Vault 存在时正确加载：

```yaml
softdepend: [Vault] # 或者depend: [Vault]
```

## 检查 Vault 和服务
在插件的 `onEnable` 方法中，检查 Vault 是否安装，并获取经济、权限和聊天服务的提供者：

```java
import net.milkbowl.vault.economy.Economy;
import net.milkbowl.vault.permission.Permission;
import net.milkbowl.vault.chat.Chat;
import org.bukkit.plugin.RegisteredServiceProvider;

public class MyPlugin extends JavaPlugin {
    private Economy economy;
    private Permission permission;
    private Chat chat;

    @Override
    public void onEnable() {
        if (getServer().getPluginManager().getPlugin("Vault") == null) {
            getLogger().info("未找到 Vault，部分功能可能被禁用。");
            return;
        }

        // 获取经济服务
        RegisteredServiceProvider<Economy> economyRsp = getServer().getServicesManager().getRegistration(Economy.class);
        if (economyRsp != null) {
            economy = economyRsp.getProvider();
        }

        // 获取权限服务
        RegisteredServiceProvider<Permission> permissionRsp = getServer().getServicesManager().getRegistration(Permission.class);
        if (permissionRsp != null) {
            permission = permissionRsp.getProvider();
        }

        // 获取聊天服务
        RegisteredServiceProvider<Chat> chatRsp = getServer().getServicesManager().getRegistration(Chat.class);
        if (chatRsp != null) {
            chat = chatRsp.getProvider();
        }
    }
}
```

## 经济服务（Economy）
经济服务允许管理玩家余额、银行账户等功能。

### 主要方法
- `boolean hasAccount(OfflinePlayer player)`：检查玩家是否拥有账户。
- `double getBalance(OfflinePlayer player)`：获取玩家余额。
- `boolean has(OfflinePlayer player, double amount)`：检查玩家是否有足够的余额。
- `EconomyResponse withdrawPlayer(OfflinePlayer player, double amount)`：从玩家账户扣款。
- `EconomyResponse depositPlayer(OfflinePlayer player, double amount)`：向玩家账户存款。

### 示例
```java
if (economy != null && economy.has(player, 10.0)) {
    EconomyResponse response = economy.withdrawPlayer(player, 10.0);
    if (response.transactionSuccess()) {
        player.sendMessage("你已支付 10 个单位。");
    } else {
        player.sendMessage("交易失败：" + response.errorMessage);
    }
} else {
    player.sendMessage("余额不足或经济服务不可用。");
}
```

## 权限服务（Permission）
权限服务用于检查和管理玩家的权限。

### 主要方法
- `boolean has(Player player, String permission)`：检查玩家是否拥有指定权限。
- `boolean playerHas(String world, OfflinePlayer player, String permission)`：检查玩家在指定世界是否拥有权限。
- `boolean playerAdd(String world, OfflinePlayer player, String permission)`：为玩家添加权限。
- `boolean playerRemove(String world, OfflinePlayer player, String permission)`：移除玩家的权限。

### 示例
```java
if (permission != null && permission.has(player, "myplugin.command")) {
    player.sendMessage("你有执行命令的权限。");
} else {
    player.sendMessage("你没有权限。");
}

// 添加权限
if (permission != null) {
    permission.playerAdd(null, player, "myplugin.newperm");
    player.sendMessage("已添加新权限。");
}
```

## 聊天服务（Chat）
聊天服务用于管理玩家和组的前缀、后缀及其他元数据。

### 主要方法
- `String getPlayerPrefix(Player player)`：获取玩家当前世界的前缀。
- `void setPlayerPrefix(Player player, String prefix)`：设置玩家当前世界的前缀。
- `String getPlayerSuffix(Player player)`：获取玩家当前世界的后缀。
- `void setPlayerSuffix(Player player, String suffix)`：设置玩家当前世界的后缀。

### 示例
```java
if (chat != null) {
    String prefix = chat.getPlayerPrefix(player);
    player.sendMessage("你的前缀是：" + prefix);
    chat.setPlayerPrefix(player, "[VIP]");
    player.sendMessage("已将你的前缀设置为 [VIP]。");
}
```

## 注意事项
- 在使用服务之前，始终检查服务提供者是否为 null，以避免空指针异常。
- 使用 `softdepend` 确保正确的加载顺序。
- 对于接受世界参数的方法，传入 null 通常表示全局或默认行为，具体取决于实现。
- 更多详细信息，请参阅 [VaultAPI Javadoc](https://milkbowl.github.io/VaultAPI/)。
