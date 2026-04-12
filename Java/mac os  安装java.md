



##### 查看JDK 安装路径
/usr/libexec/java_home -V

##### 环境变量的配置  
#首先确认当前使用的 Shell：
echo $SHELL

#如果是 zsh
vim ~/.zshrc

#JAVA_HOME 配置
export JAVA_HOME=$(/usr/libexec/java_home)

#将JAVA_HOME 添加到 PATH
export PATH=$JAVA_HOME/bin:$PATH

#根据实际 Shell 选择
source ~/.zshrc   # 或 sou  rce ~/.bash_profile

##### 验证环境

#验证 JAVA_HOME 是否设置成功
echo $JAVA_HOME

#验证 PATH 是否包含 Java
echo $PATH | grep java

#再次验证 Java 命令
java -version
javac -version