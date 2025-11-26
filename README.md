# Alfred JDK 切换器完整使用说明

## 1. 说明
通过 Alfred 快速切换本地安装的多个 JDK 版本，并让切换后的版本在 **所有终端窗口** 生效。  
JDK 8 / 11 / 17 / 21 四个版本。

## 2. 安装前准备
1. 确认本地 JDK 路径，例如 macOS 常见路径：

```
/Library/Java/JavaVirtualMachines/jdk-1.8.jdk/Contents/Home
/Library/Java/JavaVirtualMachines/jdk-11.jdk/Contents/Home
/Library/Java/JavaVirtualMachines/jdk-17.jdk/Contents/Home
/Library/Java/JavaVirtualMachines/jdk-21.jdk/Contents/Home
```

2. 创建一个统一软链接，用于 Alfred 和终端统一管理 JDK：

```bash
sudo ln -sfn /Library/Java/JavaVirtualMachines/jdk-11.jdk/Contents/Home /Library/Java/JavaVirtualMachines/CurrentJDK
```

> 这个软链接会指向当前选择的 JDK，方便 `JAVA_HOME` 统一设置。

## 3. 脚本内容（switch-jdk.sh）

```zsh
#!/bin/zsh

# 配置已安装 JDK 路径
JAVA_HOME_8="/Library/Java/JavaVirtualMachines/jdk-1.8.jdk/Contents/Home"
JAVA_HOME_11="/Library/Java/JavaVirtualMachines/jdk-11.jdk/Contents/Home"
JAVA_HOME_17="/Library/Java/JavaVirtualMachines/jdk-17.jdk/Contents/Home"
JAVA_HOME_21="/Library/Java/JavaVirtualMachines/jdk-21.jdk/Contents/Home"

# Alfred 输入参数
TARGET="{query}"

# 根据输入选择 JDK
case "$TARGET" in
  8) LINK=$JAVA_HOME_8; ICON="☕ Java 8";;
  11) LINK=$JAVA_HOME_11; ICON="🟢 Java 11";;
  17) LINK=$JAVA_HOME_17; ICON="🔵 Java 17";;
  21) LINK=$JAVA_HOME_21; ICON="🟣 Java 21";;
  *)
    cat <<EOF
{
  "items": [
    {
      "title": "❌ 输入错误: 请输入 8 / 11 / 17 / 21",
      "subtitle": "JDK Switcher",
      "arg": "",
      "valid": false
    }
  ]
}
EOF
    exit 1
    ;;
esac

# 切换软链接
sudo ln -sfn $LINK /Library/Java/JavaVirtualMachines/CurrentJDK

# 更新系统环境变量，如果已存在则替换
sed -i '' '/JAVA_HOME/d' ~/.zshrc
echo 'export JAVA_HOME=/Library/Java/JavaVirtualMachines/CurrentJDK' >> ~/.zshrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.zshrc

# 提示 Alfred 已切换
JAVA_VER=$($LINK/bin/java -version 2>&1 | head -n 1)
ESC_JAVA_VER=$(echo $JAVA_VER | sed 's/"/\"/g')

cat <<EOF
{
  "items": [
    {
      "title": "$ICON ✅ 已切换",
      "subtitle": "$ESC_JAVA_VER",
      "arg": "$ESC_JAVA_VER",
      "valid": true
    }
  ]
}
EOF
```

> ⚠️ 注意：
> - 脚本会修改 `~/.zshrc`，确保所有新终端窗口使用切换后的 JDK。
> - 如果想立即生效，请执行：
>
> ```bash
> source ~/.zshrc
> ```

## 4. Alfred Workflow 配置
JDK Switcher.alfredworkflow拖入Alfred Workflow

4. 连接 **Script Filter** 输出到 **Large Type / Notification**，显示切换结果。

## 5. 使用方法
1. 在 Alfred 输入 `jdk 11` → 切换到 JDK 11  
2. 新开终端，执行 `java -version` → 显示 JDK 11  
3. 再切换 `jdk 17` → 新终端显示 JDK 17  

## 6. 其他说明
- 脚本里使用 `sudo ln -sfn` 切换 JDK 软链接，需要输入管理员密码。  
- `.zshrc` 会追加 `JAVA_HOME`，重复切换可能会有多行，脚本已经使用 `sed` 删除旧的 `JAVA_HOME` 配置以保持整洁。  
- 如果希望切换立即在当前终端生效，可以执行：

```bash
source ~/.zshrc
```

