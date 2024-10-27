[toc]

---

# samexporter

## Missing key load_state_dict

- 报错：
  
  ```
  File "D:\software\miniconda3\Lib\site-packages\omegaconf\dictconfig.py", line 480, in _get_node
  raise ConfigKeyError(f"Missing key {key!s}")
  omegaconf.errors.ConfigAttributeError: Missing key load_state_dict
  full_key: model.load_state_dict
  object_type=dict
  ```

- 原因：没有加载到 配置文件

- 方法：
  
  ```python
  from hydra import initialize_config_module
  
  initialize_config_module("sam2_configs", version_base="1.2")
  ```


