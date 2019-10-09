# awk 常用命令

* 排除第一行,取最后一列

    awk 'NR>1 {print $NF}'