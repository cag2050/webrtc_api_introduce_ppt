title: WebRTC API 的使用介绍
speaker: 陈安国
plugins:
    - echarts

<slide class="bg-black-blue aligncenter" image="https://cn.bing.com/az/hprichbg/rb/RainierDawn_EN-AU3730494945_1920x1080.jpg .dark">

# WebRTC API 的使用介绍 {.text-landing.text-shadow}

By 陈安国 {.text-intro}

##### 快捷键：
* ←/→/空格：翻页
* ↑/↓：页面里，上移/下移
* F：全屏
* ESC：退出全屏
* -/+：总览模式/退出总览模式

* HOME/END：第一页/最后一页

[comment]: <> ([:fa-github: Github]&#40;https://github.com/cag2050/webrtc_api_introduce_ppt&#41;{.button.ghost})

<slide>
# WebRTC的历史

WebRTC（Web Real-Time Communication）是一个谷歌开源项目，它提供了一套标准API，使Web应用可以直接提供实时音视频通信功能，不再需要借助任何插件。原生通信过程采用P2P协议，数据直接在浏览器之间交互。

“为浏览器、移动平台、物联网设备提供一套用于开发功能丰富、高质量的实时音视频应用的通用协议”是WebRTC的使命。

WebRTC的发展历史如下。
* 2010年5月，谷歌收购视频会议软件公司GIPS，该公司在RTC编码方面有深厚的技术积累。
* 2011年5月，谷歌开源WebRTC项目。
* 2011年10月，W3C发布第一个WebRTC规范草案。
* 2014年7月，谷歌发布视频会议产品Hangouts，该产品使用了WebRTC技术。
* 2017年11月，WebRTC进入候选推荐标准（Candidate Recommendation，CR）阶段。

<slide>

# WebRTC的技术架构

从技术实现的角度讲，在浏览器之间进行实时通信需要使用很多技术，如音视频编解码、网络连接管理、媒体数据实时传输等，还需要提供一组易用的API给开发者使用。这些技术组合在一起，就是WebRTC技术架构，如图所示。

!![](https://res.weread.qq.com/wrepub/epub_37477429_2 .size-100.aligncenter)

<slide>

# WebRTC的网络拓扑一：Mesh网络结构

Mesh是WebRTC多方会话最简单的网络结构。<span style="color:red;">在这种结构中，每个参与者都向其他所有参与者发送媒体流，同时接收其他所有参与者发送的媒体流。</span>说这是最简单的网络结构，是因为它是Web-RTC原生支持的，无须媒体服务器的参与。Mesh网络结构如图所示。

!![](https://res.weread.qq.com/wrepub/epub_37477429_3 .size-100.aligncenter)

在Mesh网络结构中，每个参与者都以P2P的方式相互连接，数据交换基本不经过中央服务器（部分无法使用P2P的场景，会经过TURN服务器）。由于每个参与者都要为其他参与者提供独立的媒体流，因此需要N-1个上行链路和N-1个下行链路。众多上行和下行链路限制了参与人数，参与人过多会导致明显卡顿，通常只能支持6人以下的实时互动场景。

由于没有媒体服务器的参与，Mesh网络结构难以对视频做额外的处理，不支持视频录制、视频转码、视频合流等操作。

<slide>

#  WebRTC的网络拓扑二：MCU网络结构

MCU（Multipoint Control Unit）是一种传统的中心化网络结构，参与者仅与中心的MCU媒体服务器连接。MCU媒体服务器合并所有参与者的视频流，生成一个包含所有参与者画面的视频流，参与者只需要拉取合流画面，MCU网络结构如图所示。

!![](https://res.weread.qq.com/wrepub/epub_37477429_4 .size-100.aligncenter)

<span style="color:red;">这种场景下，每个参与者只需要1个上行链路和1个下行链路。</span>与Mesh网络结构相比，参与者所在的终端压力要小很多，可以支持更多人同时在线进行音视频通信，比较适合多人实时互动场景。但是MCU服务器负责所有视频编码、转码、解码、合流等复杂操作，服务器端压力较大，需要较高的配置。同时由于合流画面固定，界面布局也不够灵活。

<slide>

# WebRTC的网络拓扑三：SFU网络结构

在SFU（Selective Forwarding Unit）网络结构中，仍然有中心节点媒体服务器，但是<span style="color:red;">中心节点只负责转发，不做合流、转码等资源开销较大的媒体处理工作</span>，所以服务器的压力会小很多，服务器配置也不像MCU的要求那么高。<span style="color:red;">每个参与者需要1个上行链路和N-1个下行链路</span>，带宽消耗低于Mesh，但是高于MCU。

<span style="color:red;">我们可以将SFU服务器视为一个WebRTC参与方，它与其他所有参与方进行1对1的建立连接，并在其中起到桥梁的作用，同时转发各个参与者的媒体数据。SFU服务器具备复制媒体数据的能力，能够将一个参与者的数据转发给多个参与者。</span>SFU服务器与TURN服务器不同，TURN服务器仅仅是为WebRTC客户端提供的一种辅助数据转发通道，在无法使用P2P的情况下进行透明的数据转发，TURN服务器不具备复制、转发媒体数据的能力。

SFU对参与实时互动的人数也有一定的限制，适用于在线教学、大型会议等场景，其网络结构如图所示。

!![](https://res.weread.qq.com/wrepub/epub_37477429_5 .size-100.aligncenter)

<slide>

# WebRTC建立连接的过程：ICE建立连接

当用户A向用户B发起WebRTC呼叫时，A首先创建自己的会话描述信息（SD），我们称之为提案（offer），之后A通过信令服务器将会话描述发送给B。B同样创建自己的会话描述信息，我们称之为应答（answer），B通过信令服务器将其发送给A。这个交换过程由ICE控制，即使是在复杂的网络环境下，ICE也能确保会话描述交换顺利完成。

现在A和B都拥有了自己和对方的会话描述信息，在媒体交换格式方面达成了一致，连接成功，接下来就可以传输媒体数据了。

当会话环境发生变化（比如网络切换、更改编码格式等）时，以上过程还需要重新来过，我们将这一过程称作ICE重新协商或者ICE重启。

A和B通过信令服务器交换会话描述信息，建立网络连接的过程如图所示。

!![](https://res.weread.qq.com/wrepub/epub_37477429_29 .size-100.aligncenter)

<slide>

# WebRTC建立连接的过程：API调用

!![](https://res.weread.qq.com/wrepub/epub_37477429_30 .size-100.aligncenter)

<slide>

# STUN（Session Traversal Utilities for NAT）

位于NAT网络内的设备能够访问互联网，但并不知道NAT网络的公网IP地址，这时候就需要通过STUN协议实时发现公网IP。

STUN（Session Traversal Utilities for NAT）是一种公网地址及端口的发现协议，客户端向STUN服务发送请求，STUN服务返回客户端的公网地址及NAT网络信息。

对于建立连接的双方都位于对称NAT网络的情况，使用STUN发现网络地址后，仍然无法成功建立连接。这种情况就需要借助TURN协议提供的服务进行流量中转。

<slide>

# TURN（Traversal Using Relays around NAT）

TURN（Traversal Using Relays around NAT）通过数据转发的方式穿透NAT，解决了防火墙和对称NAT的问题。

TURN支持UDP和TCP协议。通信双方借助STUN协议能够在不使用TURN的情况下成功建立P2P连接。如有特殊情况，无法建立P2P连接，则仍需要使用TURN进行数据转发。

<slide>

# STUN 与 TURN 的区别

* 使用STUN建立的是P2P的网络模型，网络连接直接建立在通信两端，没有中间服务器介入；
* 而使用TURN建立的是流量中继的网络模型，用户两端都与TURN服务建立连接，用户的网络数据包通过TURN服务进行转发。

下图展示了单独使用STUN与结合使用STUN和TURN的对比。

!![](https://res.weread.qq.com/wrepub/epub_37477429_26 .size-100.aligncenter)

<slide>

# RTCPeerConnection接口

WebRTC使用RTCPeerConnection接口来管理对等连接，该接口提供了建立、管理、监控、关闭对等连接的方法。

```
interface RTCPeerConnection extends EventTarget {
    ...
    readonly connectionState: RTCPeerConnectionState;
    ...
    onconnectionstatechange: ((this: RTCPeerConnection, ev: Event) => any) | null;
    ...
    ontrack: ((this: RTCPeerConnection, ev: RTCTrackEvent) => any) | null;
    ...
    addTrack(track: MediaStreamTrack, ...streams: MediaStream[]): RTCRtpSender;
    addTransceiver(trackOrKind: MediaStreamTrack | string, init?: RTCRtpTransceiverInit): RTCRtpTransceiver;
    ...
    restartIce(): void;
    ...
    setLocalDescription(description?: RTCSessionDescriptionInit): Promise<void>;
    setRemoteDescription(description: RTCSessionDescriptionInit): Promise<void>;
    ...
}
```

<slide>

# RTCPeerConnection 实例化时的参数：RTCConfiguration

```
interface RTCConfiguration {
    bundlePolicy?: RTCBundlePolicy;
    certificates?: RTCCertificate[];
    iceCandidatePoolSize?: number;
    iceServers?: RTCIceServer[];
    iceTransportPolicy?: RTCIceTransportPolicy;
    rtcpMuxPolicy?: RTCRtcpMuxPolicy;
}
```

!![](https://res.weread.qq.com/wrepub/epub_37477429_34 .size-100.aligncenter)

<slide>

# 媒体流（MediaStream）

媒体流（MediaStream）是信息的载体，代表了一个媒体设备的内容流。媒体流可以被采集、传输和播放，通常一个媒体流包含多个媒体轨道，如音频轨道、视频轨道。

可以新建实例。

```
interface MediaStream extends EventTarget {
    ...
    addTrack(track: MediaStreamTrack): void;
    ...
    getAudioTracks(): MediaStreamTrack[];
    ...
    getTracks(): MediaStreamTrack[];
    getVideoTracks(): MediaStreamTrack[];
    removeTrack(track: MediaStreamTrack): void;
    ...
}
```

<slide>

# 媒体轨道（MediaStreamTrack）

媒体轨道（MediaStreamTrack）代表一个能够提供媒体服务的媒体，如音频、视频等。

不能新建实例，只能从 MediaStream 中得到。

```
interface MediaStreamTrack extends EventTarget {
    enabled: boolean;
    ...
    readonly muted: boolean;
    ...
    stop(): void;
}
```

<slide>

# 媒体流约束（MediaStreamConstraints）

```
interface MediaStreamConstraints {
    audio?: boolean | MediaTrackConstraints;
    peerIdentity?: string;
    video?: boolean | MediaTrackConstraints;
}
```

<slide>

# 媒体轨道约束（MediaTrackConstraints）

```
interface MediaTrackConstraints extends MediaTrackConstraintSet {
    advanced?: MediaTrackConstraintSet[];
}
```
```
interface MediaTrackConstraintSet {
    aspectRatio?: ConstrainDouble;
    autoGainControl?: ConstrainBoolean;
    channelCount?: ConstrainULong;
    deviceId?: ConstrainDOMString;
    echoCancellation?: ConstrainBoolean;
    facingMode?: ConstrainDOMString;
    frameRate?: ConstrainDouble;
    groupId?: ConstrainDOMString;
    height?: ConstrainULong;
    latency?: ConstrainDouble;
    noiseSuppression?: ConstrainBoolean;
    resizeMode?: ConstrainDOMString;
    sampleRate?: ConstrainULong;
    sampleSize?: ConstrainULong;
    width?: ConstrainULong;
}
```

<slide>

# 获取摄像头和麦克风

```
navigator.mediaDevices
    .getUserMedia({
        audio: true,
        video: true
    })
```

<a href="http://localhost:8081/getUserMedia" target="_blank">http://localhost:8081/getUserMedia</a>

<slide>

# 查询媒体设备

```
navigator.mediaDevices
.enumerateDevices()
```

<slide>

# 屏幕分享

```
navigator.mediaDevices
.getDisplayMedia()
```

<slide>

# 2个 RTCPeerConnection 实例进行连接的演示

<a href="http://localhost:8081/PeerConnectionCanvas" target="_blank">http://localhost:8081/PeerConnectionCanvas</a>

<slide>

# 注意点：RTCPeerConnection.addTrack() 与 MediaStream.addTrack() 的区别：
```
interface RTCPeerConnection extends EventTarget {
    addTrack(track: MediaStreamTrack, ...streams: MediaStream[]): RTCRtpSender;
}
```
```
interface MediaStream extends EventTarget {
    addTrack(track: MediaStreamTrack): void;
}
```
<slide>

# 注意点：RTCRtpSender.replaceTrack() 方法

该方法用于替换媒体流轨道。

替换媒体流轨道通常不需要进行ICE重新协商，以下场景除外。
* 新的媒体分辨率超出了现有媒体，比如新的视频分辨率更高或者更宽。
* 新的媒体帧率过高。
* 视频流轨道的预编码状态与现有轨道不同。
* 音频流轨道的通道数与现有轨道不同。
* 新的媒体源采用了硬件编码。

<slide>

# 参考资料

* 《WebRTC技术详解：从0到1构建多人视频会议系统》
* 《WebRTC音视频开发：React+Flutter+Go实战》

<slide>

# Question and Answer

<slide>

# 感谢大家
