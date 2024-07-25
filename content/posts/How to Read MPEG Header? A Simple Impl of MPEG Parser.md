---
title: "How to Read MPEG Header? A Simple Attempt to Impl a MPEG Parser"
date: 2024-07-20T17:51:06+08:00
tags:
 - MPEG
 - audio
draft: false
---
I was trying to get the sample rate of some audio data. Due to licensing issue and some other restrictions, I can't use some existing libraries to parse the MPEG header. So I decided to write a simple parser to get the sample rate from the MPEG header.

The audio data I will be mainly working about is MP3 and M4A.

### MPEG Format
MPEG is a standard for coding audio-visual information. It is mainly used for audio and video compression. The MPEG audio format is divided into 3 layers and 3 different versions. The below table shows the different versions and layers of MPEG audio.

| Version | Layer | Description |
|---------|-------|-------------|
| 1       | 1     | MPEG-1 Layer 1 |
| 1       | 2     | MPEG-1 Layer 2 |
| 1       | 3     | MPEG-1 Layer 3 |
| 2       | 1     | MPEG-2 Layer 1 |
| 2       | 2     | MPEG-2 Layer 2 |
| 2       | 3     | MPEG-2 Layer 3 |
| 2.5     | 1     | MPEG-2.5 Layer 1 |
| 2.5     | 2     | MPEG-2.5 Layer 2 |
| 2.5     | 3     | MPEG-2.5 Layer 3 |

The commonly known MP3 format is MPEG-1 Layer 3.

### MPEG Header
The tip of parsing MPEG audio header is to understand its structure:

> AAAAAAAA AAABBCCD EEEEFFGH IIJJKLMM

Below is the structure of the MPEG header:
#### A: Frame sync
**11 bits**. It is always set to `1111 1111 111` in binary. To find the frame sync bits, we should look for something like `0xFFE0` ~ `0xFFFF` in hexadecimal.

#### B: MPEG Audio version ID
**2 bits**. It indicates the MPEG version. The following table shows the version ID and the corresponding MPEG version.

| ID | MPEG Version |
|----|--------------|
| 00 | MPEG Version 2.5 |
| 01 | Reserved |
| 10 | MPEG Version 2 |
| 11 | MPEG Version 1 |

#### C: Layer description
**2 bits**. It indicates the MPEG audio layer. The following table shows the layer description and the corresponding MPEG layer.

| ID | Layer |
|----|-------|
| 00 | Reserved |
| 01 | Layer III |
| 10 | Layer II |
| 11 | Layer I |

#### D: Protection bit
**1 bit**. It shows whether the audio frame has CRC protection. If the bit is set to 0, the frame is protected by CRC.

#### E: Bitrate index
**4 bits**. Stores the bitrate index. You can find the mapping table of bitrate index to bitrate on the internet.

#### F: Sampling rate frequency index
This is the target I was looking for. The following table shows the sampling rate frequency index and the corresponding sampling rate.

| bits | MPEG 1 | MPEG 2 | MPEG 2.5 |
|------|--------|--------|----------|
| 00   | 44100  | 22050  | 11025    |
| 01   | 48000  | 24000  | 12000    |
| 10   | 32000  | 16000  | 8000     |
| 11   | Reserved | Reserved | Reserved |

#### G: Padding bit
**1 bit**. If the bit is set to 1, the frame is padded with an extra slot.

#### H: Private bit
**1 bit**. It is used for private purposes.

#### I: Channel mode
**2 bits**. The following table shows the mapping of index to channel mode.

| bits | Mode |
|------|------|
| 00   | Stereo |
| 01   | Joint stereo(Stereo) |
| 10   | Dual channel(Stereo) |
| 11   | Single channel(Mono) |

#### J: Mode extension (Only for Joint stereo)
**2 bits**.

#### K: Copyright
**1 bit**. 0 for no copyright, 1 for copyright.

#### L: Original
**1 bit**. 0 for copy, 1 for original.

#### M: Emphasis
**2 bits**.

> In conclusion, to get the sample rate for MP3 audio we need to first find the frame sync bits, next find the MPEG version ID, and then find the sampling rate frequency index.

### Example
#### Finding the Frame Sync Bits
This is the step to know where the MPEG header starts, which I think is the most important but also the biggest hurdle. As I mentioned above, the frame sync bits are always set to `1111 1111 111` in binary. So we can find the frame sync bits by looking for `0xFFE0` ~ `0xFFFF` in hexadecimal.
![alt text](../../images/finding-mpeg-header.png)
#### Finding the MPEG Version ID
In this example, the first 2 bytes of the MPEG header `FF FB`, which is `1111 1111 1111 1011` in binary. Removing the frame sync bits, we get `1 1011`. The first 2 bits are the MPEG version ID, which is `11`. According to the table above, the MPEG version is 1.
#### Finding the Sampling Rate Frequency Index
The 3rd byte of the MPEG header contains the bitrate index and the sampling rate frequency index. In this example is `0xB0`, which is `1011 0000` in binary. The sampling rate frequency index is the 5th and 6th bits, which is `00`. According to the table above, the sampling rate is 44100Hz.

### M4A
As for M4A file, the process is much simpler. All we need to do is to first find the `moov` atom, then find the `mdhd` box, and finally get the sample rate from the box.

The mdhd box contains the sample rate of the audio data. The sample rate is stored in the 15th and 16th bytes after the "mdhd" bytes in big-endian format. In the example below, the sample rate is `0x2EE0`(marked in pink), which is `12000Hz` in decimal.


![alt text](../../images/m4a-header-example.png)