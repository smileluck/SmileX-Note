[TOC]

---

# SAM

> FACEBOOK_SAM:https://github.com/facebookresearch/segment-anything?tab=readme-ov-file
> 
> MOBILE_SAM:https://github.com/ChaoningZhang/MobileSAM
> 
> FAST_SAM:

## samexporter

>  **[samexporter](https://github.com/vietanhdev/samexporter)**
> 
> 关于利用samexporter将模型导出成onnx

1. 下载并安装依赖
   
   ```shell
   git clone https://github.com/vietanhdev/samexporter
   cd samexporter
   pip install -e .
   pip install -r requirements.txt
   ```

2. 修正依赖版本（可选）
   
   ```shell
   pip uninstall onnxruntime
   pip install onnxruntime==1.15.1
   ```

3. 创建文件夹，将模型放到original_models
   
   ```shell
   mkdir original_models
   mkdir output_models
   ```

4. 转换 
   
   > Use "quantized" models for faster inference and smaller model size. However, the accuracy may be lower than the original models.
   
   ```shell
   # mobile_sam encoder
   python -m samexporter.export_encoder --checkpoint original_models/mobile_sam.pt --output output_models/mobile_sam/mobile_sam.encoder.onnx     --model-type mobile     --quantize-out output_models/mobile_sam/mobile_sam.encoder.quant.onnx     --use-preprocess
   
   # sam_vit_b_01ec64 decoder
   python -m samexporter.export_decoder --checkpoint original_models/mobile_sam.pt      --output output_models/mobile_sam/mobile_sam.decoder.onnx      --model-type mobile      --quantize-out output_models/mobile_sam/mobile_sam.decoder.quant.onnx      --return-single-mask
   
   # sam_vit_b_01ec64 encoder
   python -m samexporter.export_encoder --checkpoint original_models/sam_vit_b_01ec64.pth \
       --output output_models/sam_vit_b_01ec64.encoder.onnx \
       --model-type vit_b \
       --quantize-out output_models/sam_vit_b_01ec64.encoder.quant.onnx \
       --use-preprocess
   
   # sam_vit_b_01ec64 decoder
   python -m samexporter.export_decoder --checkpoint original_models/sam_vit_b_01ec64.pth      --output output_models/sam_vit_b_01ec64.decoder.onnx      --model-type vit_b      --quantize-out output_models/sam_vit_b_01ec64.decoder.quant.onnx      --return-single-mask
   ```
