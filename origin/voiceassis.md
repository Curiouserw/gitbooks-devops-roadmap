









ESP32-S3 通过 I2S 接口采集 16kHz/16bit 单声道 PCM 音频，数据流约为 32KB/s（即16kHz×16bit≈256kbps）。使用μ-law 等压缩将数据量减半（约16KB/s）





```bash
ffmpeg -f avfoundation -list_devices true -i ""
```





# 唤醒词模型：WakeNet



https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32s3/wake_word_engine/README.html



https://github.com/dscripka/openWakeWord





`sounddevice` ：音频采集处理

- **采样率**：16000Hz（兼顾音质与Whisper兼容性）
- **量化精度**：32位浮点（内部处理），传输时转为16位PCM
- **音频块大小**：1280样本（80ms，平衡延迟与处理效率）
- **缓冲区策略**：双缓冲队列防止数据丢失

`openwakeword`：唤醒检测（可自动训练唤醒词）

`VAD语音活动检测`：