[toc]

---

# 前言

- [官网](https://ffmpeg.org/ffprobe.html#Main-options)

# 命令示例
## 查看文件编码格式
- 输入

```shell
ffprobe -i file.mp4 -show_streams -select_streams v -print_format json 
```

- 输出

```shell
"streams": [
        {
            "index": 0,
            "codec_name": "h264",
            "codec_long_name": "H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10",
            "profile": "High",
            "codec_type": "video",
            "codec_tag_string": "avc1",
            "codec_tag": "0x31637661",
            "width": 706,
            "height": 360,
            "coded_width": 706,
            "coded_height": 360,
            "closed_captions": 0,
            "film_grain": 0,
            "has_b_frames": 2,
            "sample_aspect_ratio": "1:1",
            "display_aspect_ratio": "353:180",
            "pix_fmt": "yuv420p",
            "level": 30,
            "chroma_location": "left",
            "field_order": "progressive",
            "refs": 1,
            "is_avc": "true",
            "nal_length_size": "4",
            "id": "0x1",
            "r_frame_rate": "23857/1000",
            "avg_frame_rate": "23857/1000",
            "time_base": "1/23857",
            "start_pts": 0,
            "start_time": "0.000000",
            "duration_ts": 185000,
            "duration": "7.754537",
            "bit_rate": "882642",
            "bits_per_raw_sample": "8",
            "nb_frames": "185",
            "extradata_size": 46,
            "disposition": {
                "default": 1,
                "dub": 0,
                "original": 0,
                "comment": 0,
                "lyrics": 0,
                "karaoke": 0,
                "forced": 0,
                "hearing_impaired": 0,
                "visual_impaired": 0,
                "clean_effects": 0,
                "attached_pic": 0,
                "timed_thumbnails": 0,
                "non_diegetic": 0,
                "captions": 0,
                "descriptions": 0,
                "metadata": 0,
                "dependent": 0,
                "still_image": 0
            },
            "tags": {
                "language": "und",
                "handler_name": "VideoHandler",
                "vendor_id": "[0][0][0][0]",
                "encoder": "Lavc60.31.102 libx264"
            }
        }
    ]
}
```

