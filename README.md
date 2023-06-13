# USB-microphone-on-RPi4
Raspberry Pi に挿したUSBマイクから音声データを取り出す。 

## 1. Environment
- Linux rpi4 5.15.84-v8+ #1613 SMP PREEMPT Thu Jan 5 12:03:08 GMT 2023 aarch64 GNU/Linux
- pipenv 


## 2. preparation
### 1. usbマイクを挿入する前後で新しく認識されたUSBマイクを確認する。  

```bash
# マイク挿入前
$ lsusb
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 003: ID 0483:5750 STMicroelectronics LED badge -- mini LED display -- 11x44
Bus 001 Device 002: ID 2109:3431 VIA Labs, Inc. Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

# マイク挿入後
$ lsusb
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 003: ID 0483:5750 STMicroelectronics LED badge -- mini LED display -- 11x44
Bus 001 Device 006: ID 08bb:2902 Texas Instruments PCM2902 Audio Codec  ### <- NEW
Bus 001 Device 002: ID 2109:3431 VIA Labs, Inc. Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

### 2. USBオーディオ優先度の確認
1. 確認
```bash
$ cat /proc/asound/modules
 0 snd_bcm2835
 1 snd_bcm2835
 2 snd_usb_audio # <- mic
```  
2 snd_usb_audioがusbマイクなので、これの優先度を変更する。

2. configファイルを作成する
新たに「/etc/modprobe.d/alsa-base.conf」というファイルを作成する。
```bash
$ sudo nano /etc/modprobe.d/alsa-base.conf
```
```nano
# /etc/modprobe.d/alsa-base.conf
options snd slots=snd_usb_audio,snd_bcm2835,snd_bcm2835
options snd_usb_audio index=0
options snd_bcm2835 index=1
options snd_bcm2835 index=2
```

3. 最終確認
```bash
$ cat /proc/asound/modules
 0 snd_usb_audio  # <- mic
 1 snd_bcm2835
 2 snd_bcm2835
```

### 3. マイクで録音・再生してゲイン調整
1. マイクのサウンドカード番号の確認
```bash
$ arecord -l
**** List of CAPTURE Hardware Devices ****
card 0: Device [USB PnP Sound Device], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

2. マイクの確認のためtest.wavに出力
```bash
arecord -f S16_LE -r 44100 -Dhw:0,0 test.wav
```

3. マイクゲインの調整
```bash
$ amixer sset Mic 60 -c 0
Simple mixer control 'Mic',0
  Capabilities: cvolume cvolume-joined cswitch cswitch-joined
  Capture channels: Mono
  Limits: Capture 0 - 16
  Mono: Capture 16 [100%] [23.81dB] [on]
```

### 4. pyenvとpyaudioのインストール
1. pyaudioをインストールする準備 [参照](https://plaza.rakuten.co.jp/qualis00/diary/202105150000/)  
```bash
sudo apt-get install python-dev portaudio19-dev
```

2. pipenvで環境を管理する
