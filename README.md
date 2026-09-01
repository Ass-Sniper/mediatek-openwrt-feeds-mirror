# mediatek-openwrt-feeds-mirror
MTK 的 feed 仓库 mirror 到 GitHub



**OpenWrt 官方 feeds（telephony、packages 等）速度很快**
**唯独 MediaTek 的 mtk_openwrt_feed（git01.mediatek.com）一直非常慢**

这是所有 MTK SDK 使用者普遍遇到的问题，不是你的代理、网络、Git 设置的问题。

这是 **源站本身的结构性问题**，我给你做一个完整解释，并告诉你最佳解决办法。

---

# **原因 1：git01.mediatek.com 位于台湾/新加坡机房，带宽非常小 & 限流严重**

MediaTek 的 OpenWrt feed 仓库并不是 CDN，也不是 GitHub、GitLab 这种拥有全球加速的服务，而是：

* 自建 Git Server（Gitolite）
* 带宽通常只有几十 Mbps
* 经常限流（特别是 clone 时）
* 访问量小导致未优化
* 高峰期极其慢（10 KB/s~200 KB/s）

**这不是你本地可以解决的。全世界都慢。**

---

# **原因 2：git01.mediatek.com 对代理不友好（CDN 没法加速）**

即使你开代理：

* Clash 只能加速出国 → 但目标站本身出口带宽极弱
* 没有 CDN → 代理加速不了源站服务器性能
* MTK Git 服务对并发进行强限制

所以你看到：

```
remote: Counting objects: 47, done
remote: Finding sources: 100% (47/47)
```

**卡在 compressing / receiving objects 不动**
就是因为 MediaTek 服务端太慢，而不是你本地。

---

# **原因 3：MediaTek feed 仓库体积不大，但服务器会对 clone 做限速**

即使仅有几十个对象：

```
Cloning into './feeds/mtk_openwrt_feed'...
remote: Counting objects: 47, done
remote: Finding sources: 100% (47/47)
```

但是“Finding sources”会花很长时间，因为：

* 服务器很慢
* 对 IP / 连接数限流
* 国内访问不稳定（即使走代理）

---

# 为什么 telephony / packages 很快？

```
git.openwrt.org
github.com
kernel.googlesource.com
gitlab.com
```

这些都有：

* 全球 CDN
* 高带宽
* 高并发能力

所以你看到 telephony 只有几 MB，飞快。

---

# **实际原因总结（非常重要）**

**mtk_openwrt_feed 慢不是你的配置问题，是 MediaTek Git 服务器本身慢。
所有人都一样慢。**

Literally:
➡ Clone MTK feed 的速度取决于 MediaTek 服务器的当天心情。

---

# 解决方案（从最有效到最简单）

## **方案 1：自己在 GitHub/GitLab 上建立镜像（最推荐）**

你可以把 MTK 的 feed 仓库 mirror 到 GitHub：

```
git clone --mirror https://git01.mediatek.com/openwrt/feeds/mtk-openwrt-feeds
cd mtk-openwrt-feeds.git
git push --mirror https://github.com/你的仓库/mtk-openwrt-feeds.git
```

然后把 feeds.conf 中的：

```
src-git mtk_openwrt_feed https://git01.mediatek.com/openwrt/feeds/mtk-openwrt-feeds
```

改成：

```
src-git mtk_openwrt_feed https://github.com/你的ID/mtk-openwrt-feeds.git
```

 加速效果：**几十倍**

---

## **方案 2：使用 git 镜像工具（ghproxy / hub.fastgit / cnproxy）**

例如：

```
src-git mtk_openwrt_feed https://ghproxy.com/https://git01.mediatek.com/openwrt/feeds/mtk-openwrt-feeds
```

有时能提高一点，但不稳定。

---

##  **方案 3：使用 `git clone --depth=1`（但不适用 feed update）**

因为 feed update 需要完整仓库，这个方法有限。

---

# 建议你采用的最终操作（推荐）

我给你一键脚本，把 MTK feed 镜像到 GitHub：

```bash
git clone --mirror https://git01.mediatek.com/openwrt/feeds/mtk-openwrt-feeds
cd mtk-openwrt-feeds.git
git remote add github https://github.com/你的ID/mtk-openwrt-feeds.git
git push --mirror github
```

然后编辑 feed：

```
src-git mtk_openwrt_feed https://github.com/你的ID/mtk-openwrt-feeds.git
```

以后 update 的速度将从 **几十 KB/s → 数十 MB/s**

---
