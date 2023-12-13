# 简介
Forked From Loyalsoldier/geoip

## 与官方版 GeoIP 的区别

- 中国大陆 IPv4 地址数据融合了 [IPIP.net](https://github.com/17mon/china_ip_list/blob/master/china_ip_list.txt) 和 [@gaoyifan/china-operator-ip](https://github.com/gaoyifan/china-operator-ip/blob/ip-lists/china.txt)
- 中国大陆 IPv6 地址数据融合了 MaxMind GeoLite2 和 [@gaoyifan/china-operator-ip](https://github.com/gaoyifan/china-operator-ip/blob/ip-lists/china6.txt)
- 新增类别（方便有特殊需求的用户使用）：
  - `geoip:google`（`GEOIP,GOOGLE`）


## 定制 GeoIP 文件

可通过以下几种方式定制 GeoIP 文件：

- **在线生成**：[Fork](https://github.com/Loyalsoldier/geoip/fork) 本仓库后，修改自己仓库内的配置文件 `config.json` 和 GitHub Workflow `.github/workflows/build.yml`
- **本地生成**：
  - 安装 [Golang](https://golang.org/dl/) 和 [Git](https://git-scm.com)
  - 拉取项目代码: `git clone https://github.com/Loyalsoldier/geoip.git`
  - 进入项目根目录：`cd geoip`
  - 修改配置文件 `config.json`
  - 运行代码：`go run ./`

**特别说明：**

- **在线生成**：[Fork](https://github.com/Loyalsoldier/geoip/fork) 本项目后，如果需要使用 MaxMind GeoLite2 Country CSV 数据文件，需要在自己仓库的 **[Settings]** 选项卡的 **[Secrets]** 页面中添加一个名为 **MAXMIND_GEOLITE2_LICENSE** 的 secret，否则 GitHub Actions 会运行失败。这个 secret 的值为 MAXMIND 账号的 LICENSE KEY，需要[**注册 MAXMIND 账号**](https://www.maxmind.com/en/geolite2/signup)后，在[**个人账号管理页面**](https://www.maxmind.com/en/account)左侧边栏的 **[Services]** 项下的 **[My License Key]** 里生成。
- **本地生成**：如果需要使用 MaxMind GeoLite2 Country CSV 数据文件（`GeoLite2-Country-CSV.zip`），需要提前从 MaxMind 下载，或从本项目 [release 分支](https://github.com/Loyalsoldier/geoip/tree/release)[下载](https://github.com/Loyalsoldier/geoip/raw/release/GeoLite2-Country-CSV.zip)，并解压缩到名为 `geolite2` 的目录。

### 概念解析

本项目有两个概念：`input` 和 `output`。`input` 指数据源（data source）及其输入格式，`output` 指数据的去向（data destination）及其输出格式。CLI 的作用就是通过读取配置文件中的选项，聚合用户提供的所有数据源，去重，将其转换为目标格式，并输出到文件。

These two concepts are notable: `input` and `output`. The `input` is the data source and its input format, whereas the `output` is the destination of the converted data and its output format. What the CLI does is to aggregate all input format data, then convert them to output format and write them to GeoIP files by using the options in the config file.

### 支持的格式

关于每种格式所支持的配置选项，查看本项目 [`config-example.json`](https://github.com/Loyalsoldier/geoip/blob/HEAD/config-example.json) 文件。

支持的 `input` 输入格式：

- **text**：纯文本 IP 和 CIDR（例如：`1.1.1.1` 或 `1.0.0.0/24`）
- **private**：局域网和私有网络 CIDR（例如：`192.168.0.0/16` 和 `127.0.0.0/8`）
- **cutter**：用于裁剪前置步骤中的数据
- **v2rayGeoIPDat**：V2Ray GeoIP dat 格式（`geoip.dat`）
- **maxmindMMDB**：MaxMind mmdb 数据格式（`GeoLite2-Country.mmdb`）
- **maxmindGeoLite2CountryCSV**：MaxMind GeoLite2 country CSV 数据（`GeoLite2-Country-CSV.zip`）
- **clashRuleSetClassical**：[classical 类型的 Clash RuleSet](https://github.com/Dreamacro/clash/wiki/premium-core-features#classical)
- **clashRuleSet**：[ipcidr 类型的 Clash RuleSet](https://github.com/Dreamacro/clash/wiki/premium-core-features#ipcidr)
- **surgeRuleSet**：[Surge RuleSet](https://manual.nssurge.com/rule/ruleset.html)

支持的 `output` 输出格式：

- **text**：纯文本 CIDR（例如：`1.0.0.0/24`）
- **v2rayGeoIPDat**：V2Ray GeoIP dat 格式（`geoip.dat`，适用于 [V2Ray](https://github.com/v2fly/v2ray-core)、[Xray-core](https://github.com/XTLS/Xray-core) 和 [Trojan-Go](https://github.com/p4gefau1t/trojan-go)）
- **maxmindMMDB**：MaxMind mmdb 数据格式（`GeoLite2-Country.mmdb`，适用于 [Clash](https://github.com/Dreamacro/clash) 和 [Leaf](https://github.com/eycorsican/leaf)）
- **clashRuleSetClassical**：[classical 类型的 Clash RuleSet](https://github.com/Dreamacro/clash/wiki/premium-core-features#classical)
- **clashRuleSet**：[ipcidr 类型的 Clash RuleSet](https://github.com/Dreamacro/clash/wiki/premium-core-features#ipcidr)
- **surgeRuleSet**：[Surge RuleSet](https://manual.nssurge.com/rule/ruleset.html)

### 注意事项

由于 MaxMind mmdb 文件格式的限制，当不同列表的 IP 或 CIDR 数据有交集或重复项时，后写入的列表的 IP 或 CIDR 数据会覆盖（overwrite）之前已写入的列表的数据。譬如，IP `1.1.1.1` 同属于列表 `AU` 和列表 `Cloudflare`。如果 `Cloudflare` 在 `AU` 之后写入，则 IP `1.1.1.1` 归属于列表 `Cloudflare`。

为了确保某些指定的列表、被修改的列表一定囊括属于它的所有 IP 或 CIDR 数据，可在 `output` 输出格式为 `maxmindMMDB` 的配置中增加选项 `overwriteList`，该选项中指定的列表会在最后逐一写入，列表中最后一项优先级最高。若已设置选项 `wantedList`，则无需设置 `overwriteList`。`wantedList` 中指定的列表会在最后逐一写入，列表中最后一项优先级最高。



## License

[CC-BY-SA-4.0](https://creativecommons.org/licenses/by-sa/4.0/)

This product includes GeoLite2 data created by MaxMind, available from [MaxMind](http://www.maxmind.com).

