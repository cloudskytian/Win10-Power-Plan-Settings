# Win10 电源计划设置

## 2025/1/14 更新 Win10 应用 Win11 调度

Win11 24h2 上存在新的调度机制，其实现方式是通过应用 .ppkg 文件给 Windows 电源计划覆盖默认值起作用的（24h2 上的 AMD 鸡血补丁就是这么实现的），而该机制 Win10 下也能生效（Win10 底层不支持的调度策略会被忽略，不过基本上影响不大）

**电源计划记得选平衡，ppkg 仅适用于平衡模式，卓越和高性能已经不再适用了；反之，如果你想自己精细调节其他隐藏选项，不想使用 ppkg，那就不要选平衡，平衡会导致 ppkg 覆盖掉一些手动选项**

另：AMD 也存在大小核调度问题，AMD 会将 CPPC 最低的核心视为“小核”，推荐 AMD 的异类策略设置为 4

### Intel 部分：

将 [Power.Settings.Processor.Intel.ppkg](Power.Settings.Processor.Intel.ppkg) 复制到 C:\Windows\Provisioning\Packages ，然后在命令行执行 %windir%\system32\ProvTool.exe /turn 5 /source LogonIdleTask 即可生效

可以在装完 ppkg 后结合下面的大小核调度策略一起使用

### AMD 部分：

AMD 的芯片组驱动会自动安装最新的适用于 AMD 的 ppkg，因此直接安装 AMD 芯片组驱动即可

装好后去 Windows 设置中的电源和睡眠选项，将底部的性能和能量滑块设置为最佳性能即可（如果没有滑块，检查你的电源计划是不是平衡）

也可以结合下方大小核调度策略一起使用，不过 AMD 的默认策略已经够合理了，不是追求极限性能没必要改了

---

## Win10 上优化大小核调度策略

众所周知英特尔12代开始使用大小核策略，12600（不带k）以上的 cpu 都是P核性能核 + E核能效核的结构，而微软为了推广 win11 只在 win11 上对大小核的调度有优化，win10 上默认策略是后台程序一概跑在小核上，你开个浏览器或其他什么东西在前台，干活的程序扔在后台，这个程序就会跑到小核上运行 ~~，你摸鱼 cpu 也摸鱼~~。实际上 win10 也是支持大小核调度的，只不过默认都隐藏掉了，可以通过以下方式开启：

1. ~~先打开卓越性能电源计划（非必要，不过启用后可以更极致地压榨cpu）：~~

    ~~reg add HKLM\System\CurrentControlSet\Control\Power /v PlatformAoAcOverride /t REG_DWORD /d 0 /f~~

    ~~powercfg -duplicatescheme e9a42b02-d5df-448d-aa00-03f14749eb61~~

    ~~此时控制面板里应该已经有卓越性能了，可以手动切过去~~

    2025/1/14 更新：电源计划选平衡，ppkg 仅适用于平衡模式，卓越和高性能已经不再适用了

2. 把一些隐藏的 Windows 电源选项打开

    1. 台式机

        echo 生效的异类策略

        powercfg -attributes SUB_PROCESSOR 7f2f5cfa-f10c-4823-b5e1-e93ae85f46b5 -ATTRIB_HIDE

        echo 异类线程调度策略

        powercfg -attributes SUB_PROCESSOR 93b8b6dc-0698-4d1c-9ee4-0644e900c85d -ATTRIB_HIDE

        echo 异类短运行线程调度策略

        powercfg -attributes SUB_PROCESSOR bae08b81-2d5e-4688-ad6a-13243356654b -ATTRIB_HIDE

    2. 笔记本

        由于笔记本大多使用“现代电源计划”，cmd 或注册表无效，需要使用 [PowerSettingsExplorer](https://forums.guru3d.com/threads/windows-power-plan-settings-explorer-utility.416058/) 软件进行调节

3. 现在可以去电源计划修改以下选项：

    echo 异类线程调度策略：使用异类策略 4（AMD 适用）

    powercfg /setacvalueindex e9a42b02-d5df-448d-aa00-03f14749eb61 54533251-82be-4824-96c1-47b60b740d00 7f2f5cfa-f10c-4823-b5e1-e93ae85f46b5 004

    echo 异类线程调度策略：使用异类策略 0（Intel 适用）

    powercfg /setacvalueindex e9a42b02-d5df-448d-aa00-03f14749eb61 54533251-82be-4824-96c1-47b60b740d00 7f2f5cfa-f10c-4823-b5e1-e93ae85f46b5 000

    echo 异类短运行线程调度策略：首选高性能处理器

    powercfg /setacvalueindex e9a42b02-d5df-448d-aa00-03f14749eb61 54533251-82be-4824-96c1-47b60b740d00 93b8b6dc-0698-4d1c-9ee4-0644e900c85d 002

    echo 异类短运行线程调度策略：首选高性能处理器

    powercfg /setacvalueindex e9a42b02-d5df-448d-aa00-03f14749eb61 54533251-82be-4824-96c1-47b60b740d00 bae08b81-2d5e-4688-ad6a-13243356654b 002

    也可以去控制面板的电源计划里手动设置（还是建议手动设置一下，当检查了，万一命令失效了呢2333）

4. 现在程序应该都会优先跑在大核上了，大核满了才会转去小核运行

## 参考文献：

[哎，感觉Win11真的烂完了，SB微软阿三 NGA玩家社区](https://nga.178.com/read.php?tid=42028135)

[关于Intel/AMD/笔记本OEM是如何通过Provisioning来覆盖Windows默认电源设置的调查 NGA玩家社区](https://nga.178.com/read.php?tid=40549025)

[Windows电源设置注释 - 哔哩哔哩](https://www.bilibili.com/opus/744635123301875753)

[自用的WIndows电源设置（踩坑后的修订版） - 哔哩哔哩](https://www.bilibili.com/opus/713693474507980821)

[最简单的方式解决Intel大小核调度问题 - 电脑讨论(新) - Chiphell - 分享与交流用户体验](https://www.chiphell.com/forum.php?mod=viewthread&tid=2565551)
