# Verification

## Check conf dir
```bash
$ ls ~/.config/pipewire/
```
Expected output:
```bash
pipewire.conf.d
````

## Check pipewire conf
```bash
$ cat ~/.config/pipewire/pipewire.conf.d/10-high-res.conf
```
Expected output:
```bash
context.properties = {
    default.clock.rate = 192000
    default.clock.allowed-rates = [ 44100 48000 88200 96000 176400 192000 ]
}
```

## Check wireplumber conf
```bash
$ cat ~/.config/wireplumber/wireplumber.conf.d/51-edifier-profile.conf
```
Expected output:
```bash
monitor.alsa.rules = [
    {
        matches = [
            { device.name = "alsa_card.usb-EDIFIER_EDIFIER_S880DB_MKII-00" }
        ]
        actions = {
            update-props = {
                api.acp.auto-profile = false
                device.profile        = "output:analog-stereo"
            }
        }
    }
]
```

## Confirm PipeWire still sees 192k as default
```bash
$ pw-metadata -n settings 0 | grep -E "clock.rate|allowed-rates"
```
Expected output:
```bash
update: id:0 key:'clock.rate' value:'192000' type:''
update: id:0 key:'clock.allowed-rates' value:'[ 44100, 48000, 88200, 96000, 176400, 192000 ]' type:''
```

## Confirm Edifier is on analog-stereo
```bash
wpctl inspect @DEFAULT_AUDIO_SINK@ | grep -E "device.profile.name|node.description"
```
Expected output:
```bash
    device.profile.name = "analog-stereo"
  * node.description = "EDIFIER S880DB MKII Analog Stereo"
```

## Confirm it can run 192kHz
Simulate a sine wave (ctrl-c to stop after finish)
```bash
speaker-test -D default -r 192000 -F S24_LE -c 2 -t sine -f 440
```

Check the sound output (in another terminal)
```bash
cat /proc/asound/card6/pcm0p/sub0/hw_params
```
Expected output:
```bash
access: MMAP_INTERLEAVED
format: S32_LE
subformat: STD
channels: 2
rate: 192000 (192000/1)
period_size: 1024
buffer_size: 32768
```