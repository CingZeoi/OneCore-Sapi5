# 借助 SAPI5 接口调用 OneCore 语音

## 介绍

本项目旨在提供一种解决方案，以克服 Microsoft Windows 操作系统中 **Speech API 5 (SAPI5)** 与 **OneCore 平台语音库** 之间的兼容性限制。 默认配置下，基于 SAPI5 架构的文本转语音 (TTS) 程序无法直接访问和利用 Windows 10 及 Windows 11 系统内置的高质量 OneCore 语音，例如 "Microsoft Yaoyao Mobile" 和 "Microsoft Kangkang Mobile" 等。 本项目通过提供一系列预配置的注册表文件，**实现了将 OneCore 语音引擎整合至 SAPI5 框架，从而扩展了 SAPI5 程序可用的语音资源，并提升了 TTS 输出的质量。**

## SAPI5 与 OneCore 语音库的集成

为了充分理解本项目的价值，有必要阐明 SAPI5 和 OneCore 平台在 Windows 语音技术体系中的角色，以及两者之间存在集成困难的原因：

* **(1) Speech API 5 (SAPI5)**

SAPI5 是 Windows 系统中成熟的语音API。为开发者提供了一套标准化的工具和协议，用于在其程序中实现语音识别 (SR) 和文本转语音 (TTS) 功能。 大量传统的 TTS 软件，以及辅助技术应用，均构建于 SAPI5 架构之上。 然而，SAPI5 的设计和架构相对传统，其默认可访问的语音引擎可能未能充分集成 Windows 系统中最新的语音架构。

* **(2) OneCore 平台及其语音库**

OneCore 代表了 Microsoft Windows 操作系统架构的现代化演进。 作为 Windows 10 和 Windows 11 的统一核心平台，OneCore 整合了先进的系统组件和技术，包括高质量的语音合成引擎及语音库。 **Microsoft Mobile 语音** 即为 OneCore 平台语音库的典型代表，其特点在于音质卓越、语调自然、语言覆盖范围广泛。 这些 OneCore 语音通常应用于 Windows 系统自身的现代应用和服务，如 Cortana 语音助手和讲述人屏幕报读程序。

SAPI5 与 OneCore 语音库之间的兼容问题，导致了高质量 OneCore 语音资源在传统 SAPI5 程序中无法直接利用的现状，限制了用户在各类 TTS 应用中获得最佳语音体验。

## 通过修改注册表解锁 SAPI5 OneCore 语音

本项目提供的方案，旨在通过修改 Windows 注册表配置，**实现将 OneCore 语音引擎注册至 SAPI5 系统，从而解除默认的访问限制，并使 OneCore 语音库能够被 SAPI5 程序调用。**

* **(1) SAPI5 语音引擎注册机制**

SAPI5 系统采用基于注册表的机制来管理和识别可用的语音引擎。 当 SAPI5 运行时环境启动或程序请求可用的语音引擎列表时，系统将扫描 Windows 注册表中的特定路径：`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Speech\Voices\Tokens`。 此路径下的每一个子项均代表一个已注册的语音引擎 “令牌 (Token)”。 每个 Token 包含语音引擎的描述信息、组件标识符 (CLSID)、语音数据文件路径等关键参数。

* **(2) 基于注册表修改的解锁原理**

本项目提供的注册表文件 (.reg) 预先定义了 OneCore 语音引擎的 “令牌” 信息，并将其写入至 SAPI5 系统扫描的注册表路径 (`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Speech\Voices\Tokens`)。 通过导入这些注册表文件，系统实际上执行了以下操作：

1. **注册 OneCore 语音引擎令牌：** 在 SAPI5 注册表路径下创建新的子项，用于注册 OneCore 语音引擎的 Token。
2. **提供关键配置信息：** 为每个 Token 项配置必要的注册表值，包括：
* **CLSID 值：** 指向 OneCore 语音引擎组件的 COM 类标识符，使 SAPI5 系统能够创建和调用语音引擎对象。
* **数据文件路径：** 指定 OneCore 语音引擎的语言数据文件 (`LangDataPath`) 和语音数据文件 (`VoicePath`) 在磁盘上的位置，使 SAPI5 系统能够加载必要的语音资源。
* **属性信息：** 定义语音引擎的元数据属性（例如，语言、性别、年龄等），以便 SAPI5 程序能够识别和选择合适的语音引擎。

**通过上述注册表修改操作，SAPI5 系统将被 “告知” OneCore 语音引擎的存在，并将其纳入可用的语音引擎列表。 当 SAPI5 程序请求语音合成服务时，系统将能够识别和加载已注册的 OneCore 语音引擎，从而实现对 OneCore 语音库的调用。**

* **(3) 事先安装OneCore 语音包**

**重要的是，本解决方案依赖于 Windows 系统中已安装的 OneCore 语音资源。** 注册表文件本身 **不包含任何语音数据文件**，仅为 SAPI5 系统提供访问已安装语音资源的 “索引”。 因此，在导入注册表文件之前，用户必须 **预先安装目标 OneCore 语音引擎对应的语言包和语音组件**。

## 使用方法与操作步骤

**(1) 语音库安装准备 (先决条件)**

在执行注册表导入操作之前，务必确保已安装目标 OneCore 语音引擎所需的语言包和语音组件。 操作步骤如下：

1. **打开 "电脑设置"** 通过 Windows "开始" 菜单，访问并启动 "设置"。
2. **浏览至 "时间和语言" 设置：** 在 "设置" 窗口中，定位并选择 "时间和语言" 选项。
3. **选择 "区域和语言" 选项卡：** 在 "时间和语言" 设置界面，切换至 "区域和语言" 选项卡。
4. **添加目标语言：** 在 "语言" 部分，点击 "添加语言" 按钮，并在语言列表中选择所需的语言 (例如，中文 (简体))。
5. **下载语音资源组件：** 选中已添加的语言，点击 "选项" 按钮，然后在语言选项页面，找到 "语音" 部分，并点击 "下载" 按钮以下载和安装语音资源组件。 **务必等待语音资源下载和安装过程完成。**
6. **(可选) 系统重启：** 为确保所有组件正确加载，建议在完成语音资源安装后重启计算机。

**(2) 导入注册表文件**

1. **下载注册表文件：** 从本项目对应语言和系统架构 (32 位或 64 位) 的文件夹中，下载相应的 `.reg` 注册表文件。
2. **执行注册表合并操作：** 找到下载的 `.reg` 文件，双击运行该文件，或右键点击选择 "合并" 菜单项。 系统将弹出用户账户控制 (UAC) 提示，请授权允许注册表编辑器进行更改。
3. **确认导入：** 在注册表编辑器弹出的确认对话框中，点击 "是" 按钮，以完成注册表信息的导入。

**(3) 验证与使用**

完成注册表导入后，即可启动任何支持 SAPI5 的TTS程序 (例如 Balabolka)。 在程序的语音选择列表中，应能找到已解锁的 OneCore 语音选项 (例如 "Microsoft Yaoyao Mobile", "Microsoft Kangkang Mobile" 等)。 选择并应用 OneCore 语音，即可体验更高质量的文本转语音输出。

## 参考资料
本项目的创建和解决方案的实现，以及对 SAPI5 与 OneCore 语音库集成问题的理解，均参考了以下资源：
1. **How can I use more of the microsoft voices to SAPI? - Microsoft Community**
* **链接:** [https://answers.microsoft.com/en-us/windows/forum/all/how-can-i-use-more-of-the-microsoft-voices-to-sapi/a1ca4484-fd2a-462b-bbcd-7bfb25a89e54](https://answers.microsoft.com/en-us/windows/forum/all/how-can-i-use-more-of-the-microsoft-voices-to-sapi/a1ca4484-fd2a-462b-bbcd-7bfb25a89e54)
* **描述:** 此 Microsoft Community 论坛帖子提出了与本项目核心问题直接相关的问题，即如何在 SAPI5 程序中使用更多 Microsoft 语音。 该帖子中的解答提供了一种通过注册表修改来解锁 Microsoft 语音的方法，为本项目的解决方案提供了重要的参考和灵感。 此链接指向的讨论是理解用户需求和问题背景的重要来源。
2. **支援的语言和语音 - Microsoft 支援服务**
* **链接:** [https://support.microsoft.com/zh-tw/windows/%E9%99%84%E9%8C%84-a-%E6%94%AF%E6%8F%B4%E7%9A%84%E8%AA%9E%E8%A8%80%E5%92%8C%E8%AA%9E%E9%9F%B3-4486e345-7730-53da-fcfe-55cc64300f01](https://support.microsoft.com/zh-tw/windows/%E9%99%84%E9%8C%84-a-%E6%94%AF%E6%8F%B4%E7%9A%84%E8%AA%9E%E8%A8%80%E5%92%8C%E8%AA%9E%E9%9F%B3-4486e345-7730-53da-fcfe-55cc64300f01)
* **描述:** Microsoft 官方支持文档，详细列出了 Windows 操作系统所支持的语言和语音列表。 该文档为用户提供了关于 Windows 系统内置语音资源的权威信息，包括不同语言和地区的可用语音种类、名称等。 本项目提供的注册表文件，其目标即是解锁并启用该文档中列出的部分高质量 OneCore 语音，使其能够被 SAPI5 程序使用。 此链接是了解可用语音资源和验证解锁效果的重要参考。
3. **Balabolka - Text-To-Speech (TTS) program**
* **链接:** [https://www.cross-plus-a.com/bportable.htm](https://www.cross-plus-a.com/bportable.htm)
* **描述:** Balabolka 是一款流行的免费文本转语音 (TTS) 软件，支持 SAPI4 和 SAPI5 接口。 本项目在开发和测试过程中，Balabolka 被用作主要的 SAPI5 程序示例，用于验证注册表修改方案的有效性，并测试解锁后的 OneCore 语音在实际 TTS 应用中的表现。 此链接指向 Balabolka 软件的官方下载页面，用户可以下载该软件以验证本项目提供的解决方案。
