## 弃用文件 /etc/sysctl.conf



从 207 版本开始，`systemd` 将不再应用 `/etc/sysctl.conf` 中的配置：它将仅应用 `/etc/sysctl.d/*` 中的配置。由于 `procps-ng` 提供的 `/etc/sysctl.conf` 中的配置已经成为内核默认设置，我们决定弃用该文件。

当升级到 `procps-ng-3.3.8-3` 时，您将会收到需要将对 `/etc/sysctl.conf` 的所有更改移动到 `/etc/sysctl.d/` 目录下的提示。最简单的方法是运行：

```
pacman -Syu
mv /etc/sysctl.conf.pacsave /etc/sysctl.d/99-sysctl.conf
```

如果您从未对 `/etc/sysctl.conf` 做任何更改，则无需采取任何操作。