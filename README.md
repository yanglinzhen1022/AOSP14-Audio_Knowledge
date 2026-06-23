# AOSP14 Audio Knowledge Base

Android 14 (AAOS) 音频子系统深度解析知识库，共15篇，覆盖从架构总论到调试与OEM定制的完整体系。

---

## 目录

| 序号 | 文件 | 主题 | 关键类/流程 |
|------|------|------|-------------|
| 01 | [01_Architecture_Overview.md](01_Architecture_Overview.md) | 架构总论 | 7层分层图、5大状态机、Binder IPC地图 |
| 02 | [02_Application_Layer.md](02_Application_Layer.md) | 应用层API | AudioTrack/Record时序、AAudio MMAP、MediaPlayer/NuPlayer、MediaCodec/Extractor/MetadataRetriever |
| 03 | [03_Java_Framework_Layer.md](03_Java_Framework_Layer.md) | Java框架层 | AudioService、MFC焦点栈、PAM duck/fade/mute、AudioMode状态机 |
| 04 | [04_Native_Framework_Layer.md](04_Native_Framework_Layer.md) | Native框架层 | libaudioclient Binder IPC、共享内存、JNI桥接 |
| 05 | [05_AudioFlinger.md](05_AudioFlinger.md) | AudioFlinger引擎 | Thread体系、FastMixer、createTrack时序、AudioMixer引擎 |
| 06 | [06_Audio_Policy_Engine.md](06_Audio_Policy_Engine.md) | 音频策略引擎 | getOutputForAttrInt、ProductStrategy、VolumeGroup曲线 |
| 07 | [07_Effects_Framework.md](07_Effects_Framework.md) | 音效框架 | EffectChain process_l、Insert/Auxiliary、Spatializer |
| 08 | [08_HAL_Layer.md](08_HAL_Layer.md) | HAL抽象层 | AIDL IModule 8子模块、StreamDescriptor、AudioGain模型 |
| 09 | [09_AAOS_Car_Audio.md](09_AAOS_Car_Audio.md) | 车载音频系统 | CarAudioService、多Zone架构、交互矩阵、CarVolumeGroup |
| 10 | [10_AudioControl_HAL.md](10_AudioControl_HAL.md) | 车载音频控制HAL | evaluateFocusRequest、Ducking/Muting流程 |
| 11 | [11_Vendor_Layer.md](11_Vendor_Layer.md) | Vendor定制层 | audio_policy_configuration.xml、car_audio_configuration.xml |
| 12 | [12_Audio_Focus_Deep_Dive.md](12_Audio_Focus_Deep_Dive.md) | Audio Focus深度解析 | Focus栈模型、duck/fade/mute执行、通话Muting |
| 13 | [13_Volume_Device_Deep_Dive.md](13_Volume_Device_Deep_Dive.md) | Volume & Device深度解析 | Volume状态机、SoundDose、Device路由迁移、USB/HDMI管理 |
| 14 | [14_Bluetooth_Audio.md](14_Bluetooth_Audio.md) | 蓝牙音频深度解析 | A2DP Codec/Offload、LE Audio VCP/Broadcast、SCO/HFP状态机 |
| 15 | [15_Debug_and_OEM_Guide.md](15_Debug_and_OEM_Guide.md) | 调试方法与OEM定制指南 | dumpsys audio、logcat过滤、问题定位、OEM定制点 |

---

## 架构分层总览

```
┌──────────────────────────────────────────────────────────────┐
│                   Layer 7: Application                       │
│   AudioTrack / AudioRecord / AAudio / MediaPlayer / ExoPlayer│
├──────────────────────────────────────────────────────────────┤
│                 Layer 6: Java Framework                       │
│   AudioService / MediaFocusControl / PlaybackActivityMonitor │
│   AudioDeviceBroker / AudioMode(6种模式) / Capture仲裁(5层) │
├──────────────────────────────────────────────────────────────┤
│                Layer 5: Native Framework                      │
│   libaudioclient(Binder IPC) / AudioSystem(路由缓存)          │
├──────────────────────────────────────────────────────────────┤
│                  Layer 4: AudioFlinger                        │
│   Thread体系 / AudioMixer+Resampler / PatchPanel / Track     │
├──────────────────────────────────────────────────────────────┤
│              Layer 3: Audio Policy Engine                     │
│   AudioPolicyManager / ProductStrategy / VolumeGroup / Engine │
├──────────────────────────────────────────────────────────────┤
│              Layer 2: Effects Framework                       │
│   EffectChain / EffectModule / Spatializer                   │
├──────────────────────────────────────────────────────────────┤
│              Layer 1: HAL + Vendor + AAOS                     │
│   Audio HAL(AIDL/HIDL) / AudioControl HAL / Config / BT HAL │
└──────────────────────────────────────────────────────────────┘
```

---

## 阅读建议

- **入门路径**: 01 → 02 → 03 → 05 → 08
- **4大状态机**: 03(AudioMode) → 12(Focus) → 13(Volume+Device)
- **策略与路由**: 01 → 06 → 13 → 08
- **车载专项**: 01 → 09 → 10 → 11 → 15
- **蓝牙专项**: 02 → 03 → 14
- **全栈调用链**: 05(播放/录音) → 12(Focus链路) → 13(Volume/Device链路)
- **调试定位**: 05 → 15

---

> 本知识库基于 Android 14 (AOSP) 源码分析，适用于 AAOS 车载音频系统开发与定制。