# JAVA AES 256 加解密 Illegal key size

出现这个的原因在于:

JAVA默认支持AES 128 Bit 的key, 如果你使用 256 Bit key, java complier 会抛出 Illegal key size Exception

有两种解决办法

第一种:替换jar

把里面的两个jar包：local_policy.jar 和 US_export_policy.jar 替换掉原来 Jdk 安装目录 $\Java\jre{6|7|8}\lib\security 下的两个jar 包接可以了

jar地址:

jdk8  https://pan.baidu.com/s/12-7IbEzDAuXGClRkrg3M8g   提取码:2f87

第二种:升级jdk版本

java 版本在 1.8.0_161之后就没有这个问题了，默认是支持,升级到1.8.0_161及更高版本