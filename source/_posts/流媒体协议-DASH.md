---
toc: true
title: 流媒体协议-DASH
date: 2024-01-05 10:11:51
pic: https://greenhaha.oss-cn-beijing.aliyuncs.com/frontend/assets/img/dash.png
tags:
  -
  - 流媒体
  - 直播
category:
  - 知识剖析
---

# 流媒体协议-DASH

## 简介

除了前面讲的 Apple 的 HLS，还有 Adobe HTTP Dynamic Streaming (HDS)、Microsoft Smooth Streaming (MSS)。他们各家的协议原理大致相同，但是格式又不一样，也无法兼容，所以 Moving Picture Expert Group (MPEG) 就把大家叫到了一起，呼吁大家一起来制定一个标准的，然后就有了[MPEG-DASH](https://www.encoding.com/mpeg-dash/),它的主要目标是形成 IP 网络承载单一格式的流媒体并提供高效与高质量服务的统一方案，解决多制式传输方案(HTTP Live Streaming, Microsoft Smooth Streaming, HTTP Dynamic Streaming)并存格局下的存储与服务能力浪费、运营高成本与复杂度、系统间互操作弱等问题。

[DASH(MPEG-DASH)](https://mpeg.chiariglione.org/standards/mpeg-dash/)全称为 Dynamic Adaptive Streaming over HTTP.是由 MPEG 和 ISO 批准的独立于供应商的国际标准，它是一种基于 HTTP 的使用 TCP 传输协议的流媒体传输技术。它诞生的目的是为了统一标准，因此是兼容 SmoothStreaming 和 HLS 的.同时支持 TS profile 和 ISO profile，支持节目观看等级控制，支持父母锁. mpeg dash 支持的 DRM 类型包括 PlayReady 和 Marlin，而 HLS 支持的是 AES128（密钥长度为 128 位的高级加密标准 Advanced Encryption Standard）加密类型。

MPEG-DASH 是一种自适应比特率流技术，可根据实时网络状况实现动态自适应下载。和 HLS, HDS 技术类似， 都是把视频分割成一小段一小段， 通过 HTTP 协议进行传输，客户端得到之后进行播放；不同的是 MPEG-DASH 支持 MPEG-2 TS、MP4(最新的 HLS 也支持了 MP4)等多种格式, 可以将视频按照多种编码切割, 下载下来的媒体格式既可以是 ts 文件也可以是 mp4 文件，MPEG—DASH 技术与编解码器无关，可使用 H.265，H.264，VP9 等任何编解码器进行编码。

安卓平台上的 ExoPlayer 支持 MPEG-DASH。另外，三星、索尼、飞利浦、松下的一些较新型号的智能电视支持 MPEG—DASH。Google 的 Chromecast、YouTube 和 Netflix 也已支持 MPEG-DASH。

![dash_compare](https://raw.githubusercontent.com/CharonChui/Pictures/master/dash_compare.png)

HLS 在 16 年支持了 fmp4，在 17 年支持了 4K。

### 思想

![dash_main_idea](https://raw.githubusercontent.com/CharonChui/Pictures/master/dash_main_idea.png)

DASH 的核心思想是，在服务端，视频被提前编好多种码率，并且被切成固定长度的视频片段，存放到 HTTP 服务器中，当客户端播放时，通过 HTTP 请求向服务器请求视频切片，并根据网络状况的变化，请求相应质量的视频切片，从而达到对网络带宽的最大利用，并且保证播放流畅。可以实现不同画质内容无缝切换。所以在 YouTube 切换画质时完全不会黑屏，更不会影响观看。

![image-20200406163508642](https://raw.githubusercontent.com/CharonChui/Pictures/master/a_dash_scenario.png)

DASH 的整个流程:

内容生成服务器(编码模块、封装模块) -> 流媒体服务器(MPD、媒体文件、HTTP 服务器) <-> DASH 客户端(控制引擎、媒体引擎、HTTP 接入容器)

![image-20200406164238038](https://raw.githubusercontent.com/CharonChui/Pictures/master/mpd_hierarchical_data.png)

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<MPD id="0564e940-122b-42bb-9d56-98f3def67247" profiles="urn:mpeg:dash:profile:isoff-main:2011" type="static" availabilityStartTime="2016-01-14T09:30:35.000Z" publishTime="2016-01-14T09:31:33.000Z" mediaPresentationDuration="P0Y0M0DT0H2M17.000S" minBufferTime="P0Y0M0DT0H0M1.000S" bitmovin:version="1.6.0" xmlns:ns2="http://www.w3.org/1999/xlink" xmlns="urn:mpeg:dash:schema:mpd:2011" xmlns:bitmovin="http://www.bitmovin.net/mpd/2015">
    <Period>
        <AdaptationSet mimeType="video/mp4" codecs="avc1.42c00d">
            <SegmentTemplate media="../video/$RepresentationID$/dash/segment_$Number$.m4s" initialization="../video/$RepresentationID$/dash/init.mp4" duration="120119" startNumber="0" timescale="30000"/>
            <Representation id="1920_9000000" bandwidth="9000000" width="3840" height="1920" frameRate="30"/>
            <Representation id="1080_5000000" bandwidth="5000000" width="2160" height="1080" frameRate="30"/>
            <Representation id="720_3000000" bandwidth="3000000" width="1440" height="720" frameRate="30"/>
            <Representation id="540_1500000" bandwidth="1500000" width="1080" height="540" frameRate="30"/>
            <Representation id="360_1000000" bandwidth="1000000" width="720" height="360" frameRate="30"/>
        </AdaptationSet>
        <AdaptationSet lang="en" mimeType="audio/mp4" codecs="mp4a.40.2" bitmovin:label="english stereo">
            <AudioChannelConfiguration schemeIdUri="urn:mpeg:dash:23003:3:audio_channel_configuration:2011" value="2"/>
            <SegmentTemplate media="../audio/$RepresentationID$/dash/segment_$Number$.m4s" initialization="../audio/$RepresentationID$/dash/init.mp4" duration="191472" startNumber="0" timescale="48000"/>
            <Representation id="1_stereo_128000" bandwidth="128000" audioSamplingRate="48000"/>
        </AdaptationSet>
    </Period>
</MPD>

```

#### MPD

DASH 采用 3GPP AHS 中定义的 MPD(Media Presentation Description)作为媒体文件的描述文件（manifest），作用类似 HLS 的 m3u8 文件。MPD 文件以 XML 格式组织，用于描述 segment 的信息，比如时间、url、视频分辨率、码率等。  
DASH 会通过 media presentation description (MPD)将视频内容切片成一个很短的文件片段，每个切片都有多个不同的码率，DASH Client 可以根据网络的情况选择一个码率进行播放，支持在不同码率之间无缝切换。
DASH 中的重要概念

#### Period

标注了视频的时长信息，也可以看做是更新 mpd 文件的最长时长，MPD 文件包含一个或者多个片段(Period)，它表示时间轴上的一段时间，每个片段都有一个起始时间和结束时间，并且包含了一个或者多个适配集合(Adaptation Set)。每个适配集合提供了一个或者多个媒体组件的信息，并包含了多种不同的码率。每个适配集合又是由多个呈现(Representation)组成，每个呈现就是同一个视频的不同特征的版本，如码率、分辨率等特征。由于每个的视频都要被切成固定长度的切片，因此每个呈现包括多个视频切片(Segment)，每个视频切片都有一个 URL 地址，这样客户端就可以通过这个地址向服务器发送 HTTP GET 请求获取该片段。同一个 Period 内，意味着可用的媒体内容及其各个可用码率（Representation）不会发生变更。直播情况下，“可能”需要周期地去服务器更新 MPD 文件，服务器可能会移除旧的已经过时的 Period,或是添加新的 Period。新的 Period 中可能会添加新的可用码率或去掉上一个 Period 中存在的某些码率(Representation)。

#### Adaptation Set

包含了媒体呈现的形式，（视频/音频/字幕）。 一个 Period 由一个或者多个 Adaptationset 组成。Adaptationset 由一组可供切换的不同码率的码流（Representation)组成，这些码流中可能包含一个（ISO profile)或者多个(TS profile)media content components，因为 ISO profile 的 mp4 或者 fmp4 segment 中通常只含有一个视频或者音频内容，而 TS profile 中的 TS segment 同时含有视频和音频内容，当同时含有多个 media component content 时，每个被复用的 media content component 将被单独描述

#### Representation

包含不同的码率、编码方式、帧率信息等。每个 Adaptationset 包含了一个或者多个 Representations,一个 Representation 包含一个或者多个 media streams，每个 media stream 对应一个 media content component。为了适应不同的网络带宽，实际播放的时候，视频会在一个 AdaptationSet 中的不同 Representaiton 之间切换码率，可能会从一个 Representation 切换到另外一个 Representation，会依次请求该 Representaiton 下不同 Segment 序列。

#### Segments

每一个具体的片段。（1,2,4,6,10s …） Segments 可以包含任何媒体数据，关于容器，官方提供了两种建议: ISO base media file format(比如 MP4 文件格式)和 MPEG-2 Transport Stream。

每个 Representation 会划分为多个 Segment。Segment 分为 4 类，其中，最重要的是：Initialization Segment（每个 Representation 都包含 1 个 Init Seg），Media Segment（每个 Representation 的媒体内容包含若干 Media Seg）

- Initialization Segment：

  Representation 的 Segments 一般都采用 1 个 Init Segment+多个普通 Segment 的方式，还有一种形式就是 Self Initialize Segment，这种形式没有单独的 Init Segment，初始化信息包括在了各个 Segment 中。Init Segment 中包含了解封装需要的全部信息，比如 Representation 中有哪些音视频流，各自的编码格式及参数。对于 ISO profile 来说(容器为 MP4)，包含了 moov box,H264 的 sps/pps 数据等关键信息存放于此（avCc box）。另外，同一个 Adaptation set 的多个 Representation 还可能共享同一个 Init Segment，该种情况下，对于 ISO profile 来说，诸如 stsd box，avCc box 等重要的 box 会含有多个 entry，每个 entry 对应一个 Representation，第一个 entry 对应第一个 Representation，第二个 entry 对应第二个 Representation，以此类推。

- Subsegment

  Segment 可能进一步划分为 subsegment，每个 subsegment 由数个 Acess Unit 组成，Segment index 提供了 subsegment 相对于 Segment 的字节范围和 presentation time range 。客户端可以先下载 Segment index。

  `*_init.mp4`: 初始的 mp4 文件，相当于视频头，在这个头文件中包含了完整的视频元信息(moov)，具体的可以使用 `MP4Box  -info` 查看。

  `*.m4s`: 即上面提到的 Segments 文件，每个 m4s 仅包含媒体信息 (moof + mdat)，而播放器是不能直接播放这个文件的，需要用支持 DASH 的播放器从 init 文件开始播放。

确切的说，当 Adaptation set 的属性@segmentAlignment 为真（true）时，同一个 Adaptation set 中的多个 Representation 之中的媒体段是对齐的，因此，当从一个 Representation A 切换到另一个 Representation B 时，若 Representation A 的第 N 个媒体段已经下载完成，切换时可直接下载 Representation B 的第 N+1 个媒体段。

DASH 对媒体段定义了三种方式：

BaseURL：单段表示

SegmentList：段列表

SegmentTemplate：段模板

单段表示是最简单的：每个 Representation 只有一个媒体段。用 BaseURL 表示。举例如下：

#### SAP 和无缝切换以及 SEEK

SAP：Stream Acess Point，可以简单理解为 I 帧，每个 Segment 的第一个帧都是 SAP，因此 Seek 时可直接 Seek 到某一个 Segment 的起始位置，利用 Init Segment+Seek 到的某个 Segment 的数据，在解封装后可实现完美解码。一般来说，同一个 Adaptation set 中的多个 Representation 是 Segment Align 的（当 Adaptation set 的属性@segmentAlignment 不为 false 时），因此，当从 Representation A 切换到 Representation B 时，如果当前 Representation A 的第 N 个 Segment 已经下载完成，切换时直接下载 Representation B 的第 N+1 个 Segment 即可。

码率自适应切换算法

- 基于带宽的码率自适应切换算法
- 基于缓存的码率自适应切换算法

![image-20200406165443536](https://raw.githubusercontent.com/CharonChui/Pictures/master/dash_change_compare.png)

此基于时间缓存的码率自适应算法对网络带宽变化反应敏感，能够有效的提高平均码率，但同时码率切换次数过大，尤其是在网络状况波动很大的情况下，这势必会造成用户体验的下降。

- MSS 拥有最高的平均码率和较少的切换次数

- HLS 的切换次数最少，但是以最低的平均码率作为代价

- HDS 不能保证流程的播放

- DASH 有足够的竞争力，也具有巨大的提升空间。

### 为什么使用 DASH

- DASH 支持多种编码，支持 H.265、H.264、VP9 等。

- DASH 支持 MultiDRM，支持 PlayReady、Widewine，采用通用加密技术，支持终端自带 DRM，可以大幅度降低 DRM 投资成本。

- DASH 支持多种文件封装，支持 MPEG-4、MPEG-2 TS。

- DASH 支持多种 CDN 对接，采用相同的封装描述对接多厂家 CDN。

- DASH 支持直播、点播、录制等丰富的视频特性。

- DASH 支持动态码率适配 Adaptive Bitrate (ABR) ，支持多码率平滑切换。

- DASH 支持缩略型描述以支持快速启动。

#### HLS vs DASH

- 在标准 HTTP 服务器上的用法： HLS 和 DASH 均可在常规 HTTP 服务器（例如 Nginx，Apache 等）上使用。
- 多个音频通道: 特别是对于多语言内容，重要的是能够在各个语言的不同音频通道之间进行切换。 DASH 和 HLS 都可以做到这一点。
- 字幕和标题: 为了给视频添加字幕，通常创建一个单独的文件，例如，文件可以具有 WebVTT 格式。然后从清单（即.m3u8 或.mpd 文件）中引用该文件。
- 插入广告: 通常，可以在 HLS 和 DASH 的实时流中插入广告。为此，只需交换单个视频块。 DASH 为此提供了一种有效的方法：标准化的界面允许有效地插入广告。
- 快速频道切换: 您可以在各个通道之间切换的速度取决于最大的子段（块）。块越小，通道更改速度越快。正如引言中已经提到的，HLS 块通常长约 10 秒，而 DASH 块通常长 2 至 4 秒。因此，DASH 在这方面领先一步。小块还具有降低代码效率的缺点。具有较小块的播放列表必须比具有较大块的播放列表更频繁地更新。这意味着包含较短视频片段的播放列表必须通过 HTTP 更频繁地更新。

## **结构与编码**

MPEG-DASH 支持 TS 和 MP4 / ISO BMFF 媒体段。HLS 只支持 MPEG-2 TS。DASH 媒体段通常比 HLS 短，2 至 4 秒比较常见。DASH 不需要特定的编解码器。视频可以使用 H264 编码，也可以用其他编码，VP9 和 H265 也是比较受欢迎的编码。

一般而言，与 HLS 相比，DASH 可以提供实质上更低的端对端延迟。这对于现场直播的工作流程很重要。此外， MPEG-DASH 的基于模板的 MPD 不需要更新，可以在网络边缘服务器进行缓存，HLS 则需要周期性地更新传播多次。

DASH 支持索引和基于时间的模版，播放器能够基于公开的时钟，如 NTPS，进行同步。这对于多相机的情况下，多个播放器之间同步会比较容易。

## **DRM**

DASH 和 HLS 之间的另一个关键区别是它支持 DRM。可是，在 DASH 中不存在一个单一通用的 DRM 解决方案。例如，Google 的 Chrome 支持 Widevine，而 Microsoft 的 Internet Explorer 支持 PlayReady。然而，通过使用 MPEG-CENC（MPEG 通用加密）结合加密媒体扩展（EME），视频流内容可以仅被加密一次。HLS 支持 AES-128 加密，以及苹果自己的 DRM，Fairplay。

#### 测试流

- HEVC HLS with fMP4: [http://bitmovin-a.akamaihd.net/content/dataset/multi-codec/hevc/stream_fmp4.m3u8](https://bitmovin-a.akamaihd.net/content/dataset/multi-codec/hevc/stream_fmp4.m3u8)

- HEVC HLS with TS (not supported by Apple): [http://bitmovin-a.akamaihd.net/content/dataset/multi-codec/hevc/stream_ts.m3u8](https://bitmovin-a.akamaihd.net/content/dataset/multi-codec/hevc/stream_ts.m3u8)

  [https://bitmovin-a.akamaihd.net/content/playhouse-vr/m3u8s/105560.m3u8](https://bitmovin-a.akamaihd.net/content/playhouse-vr/m3u8s/105560.m3u8)

- HEVC MPEG-DASH: [http://bitmovin-a.akamaihd.net/content/dataset/multi-codec/hevc/stream.mpd](https://bitmovin-a.akamaihd.net/content/dataset/multi-codec/hevc/stream.mpd)

- Multi-Codec MPEG-DASH (AVC/H.264, HEVC/H.265, VP9): http://bitmovin-a.akamaihd.net/content/dataset/multi-codec/stream.mpd

  https://bitmovin-a.akamaihd.net/content/playhouse-vr/mpds/105560.mpd

- VP9 MPEG-DASH: http://bitmovin-a.akamaihd.net/content/dataset/multi-codec/stream_vp9.mpd

参考:

- [HLS，MPEG-DASH - What is ABR?](http://telestreamblog.telestream.net/2017/05/what-is-abr/)
- [B 站我们为什么使用 DASH](https://www.bilibili.com/read/cv855111)
- [Adaptive HTTP Streaming Technologies: HLS vs. DASH](https://strivecast.io/hls-vs-mpeg-dash/)
- [自适应流媒体传输](https://blog.csdn.net/nonmarking/article/details/86351147)

在线测试播放器:

http://demo.theoplayer.com/test-your-stream-with-statistics
