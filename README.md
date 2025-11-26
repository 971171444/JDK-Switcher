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

2. 环境变量：
echo 'export JAVA_HOME=/Library/Java/JavaVirtualMachines/CurrentJDK' >> ~/.zshrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.zshrc
source ~/.zshrc

## 3. 脚本内容

```#!/bin/zsh

JAVA_HOME_8="/Library/Java/JavaVirtualMachines/jdk-1.8.jdk/Contents/Home"
JAVA_HOME_11="/Library/Java/JavaVirtualMachines/jdk-11.jdk/Contents/Home"
JAVA_HOME_17="/Library/Java/JavaVirtualMachines/jdk-17.jdk/Contents/Home"
JAVA_HOME_21="/Library/Java/JavaVirtualMachines/jdk-21.jdk/Contents/Home"

TARGET="{query}"

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
export JAVA_HOME=/Library/Java/JavaVirtualMachines/CurrentJDK
export PATH=$JAVA_HOME/bin:$PATH

JAVA_VER=$($JAVA_HOME/bin/java -version 2>&1 | head -n 1)
ESC_JAVA_VER=$(echo $JAVA_VER | sed 's/"/\\"/g')  # 转义双引号

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

## 5. 使用方法
1. 在 Alfred 输入 `jdk 11` → 切换到 JDK 11  
2. 新开终端，执行 `java -version` → 显示 JDK 11  
3. 再切换 `jdk 17` → 新终端显示 JDK 17  


