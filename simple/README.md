# Ultra 一年免费 · 极简版

一个文件，两三百行，做一件事：让 Google One 把 Ultra 的 12 个月免费试用亮出来，确认 0 元，开好。

脚本不会替你省掉绑卡这一步，也不会碰你的账号设置。它只负责把那个 UI 上看不到的 offer 拿出来。

## 一、准备

**手机**：root 过的安卓机，USB 连着电脑。Magisk、KernelSU、APatch 都行，模拟器也行。

**电脑**：Python 3.9 以上，装一个包

```bash
pip3 install frida
```

**adb**：Android SDK 里的 platform-tools。装好后执行 `adb devices`，能看到一串设备号就对了。看不到的话，在手机上打开「开发者选项 → USB 调试」，插上后点允许。

**手机上的 frida-server**：这个不在脚本里，要自己放一次。

```bash
# 电脑上和 frida 同版本的 server
curl -LO https://github.com/frida/frida/releases/download/16.5.9/frida-server-16.5.9-android-arm64.xz
xz -d frida-server-16.5.9-android-arm64.xz
adb push frida-server-16.5.9-android-arm64 /data/local/tmp/frida-server
adb shell "su -c 'chmod 755 /data/local/tmp/frida-server'"
adb shell "su -c '/data/local/tmp/frida-server &'"
```

版本号换成跟你电脑上 `frida --version` 一样的那个，对不上会连不上。

## 二、先检查环境

```bash
python3 ultra.py --check
```

它会依次看：adb、设备、root、代理、Clash、frida-server、计费模块、frida 版本。

全绿就往下走。红的那一行会直接告诉你缺什么。

## 三、手机上做两件事

脚本不代劳，因为这两步手动做比自动化靠谱。

**1. 把号加进去，Play 里切过去**

设置 → 账号 → 添加账号 → Google。加完打开 Play 商店，点右上角头像，切到这个号。

**2. 绑一张卡**

Play → 头像 → 付款和订阅 → 付款方式 → 添加信用卡。

订阅必须有付款方式，没有卡的话 Play 会直接拒。每台手机绑一次就够，后面换号共用这张卡。

## 四、开跑

```bash
python3 ultra.py
```

它会做四件事：

1. 把手机里所有 Google 账号列出来，停住等你按回车。**看一眼，不是要开的号就 Ctrl+C，回 Play 右上角切过去再跑**
2. 启动 Google One，把一个小模块注入进去
3. 查 `g1.play.20tb` 的 offer，挑免费期最长的那个
4. 拉起 Play 收银台，盯住金额

到第四步会停一下，屏幕上弹出 Play 的订阅确认页。脚本会在两种情况下自己处理：

- 看到「免费试用 12 个月」或者 `$0` → 点订阅
- 看到「您不符合免费试用条件」 → 关掉，什么都不点

有时候 Play 会让你重新输一次密码。把密码带上就跑：

```bash
python3 ultra.py --password 你的密码
```

不想每次都按回车的话，加 `--yes`：

```bash
python3 ultra.py --yes
```

## 五、确认结果

开完之后，脚本会自动打开 Play 的订阅页。

正常的结果长这样：

```
Google One
即将扣款    开始日期：2027年11月17日
            US$99.99/个月
主要支付方式 Mastercard-7558
```

明年才扣款，说明这 12 个月是免费的。Google One App 首页也会变成「当前方案：Google AI Ultra，20 TB」。

## 常见问题

**提示没有 root**

脚本要往应用进程里注入代码，没 root 做不了。

**一直读到空账号**

Play 还没加载完。先手动打开一次 Play，等首页出来，再跑。

**收银台一闪就没了**

重跑一次。连续两次都不行，检查 `adb shell settings get global http_proxy`，结果不是 `null` 就清掉。Clash 这类 VPN 会改 DNS，Play 连不上就会这样。

**提示「不符合免费试用条件」**

这个号没资格，换一个。一个号基本只有一次机会：试用一旦发起过，哪怕你没确认，offer 也会消失。所以别拿主力号试。

**收银台显示 US$99.99**

说明 offer 没选中，脚本会自动放弃，不会扣钱。

## 开完之后

到期前去 Play 取消订阅，不然会扣 $99.99。路径：Play → 头像 → 付款和订阅 → 订阅 → Google One → 取消订阅。

取消不影响已经用掉的时间，Ultra 会一直用到试用结束。
