+++
title="Cannot load onnxruntime.capi. Error: DLL load failed"
date=2020-12-17

[taxonomies]
categories=["Programming"]
tags=["onnx", "machine learning"]
+++

安装`onnxruntime-gpu`，但是运行时报错：
```
>>> import onnxruntime as ort
.venv\lib\site-packages\onnxruntime\capi\_pybind_state.py:14: UserWarning: Cannot load onnxruntime.capi. Error: 'DLL load failed: 找不到指定的模块。'.
  warnings.warn("Cannot load onnxruntime.capi. Error: '{0}'.".format(str(e)))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File ".venv\lib\site-packages\onnxruntime\__init__.py", line 13, in <module>
    from onnxruntime.capi._pybind_state import get_all_providers, get_available_providers, get_device, set_seed, \
ImportError: cannot import name 'get_all_providers' from 'onnxruntime.capi._pybind_state' (.venv\lib\site-packages\onnxruntime\capi\_pybind_state.py)
```

猜测原因可能是 `CUDA`版本不匹配。

到`onnxruntime`的`releases`页面可以查看所有信息，总结对应关系如下：

- onnx runtime v1.5.1 ~ v1.6.0 对应 cuda 10.2 ( cuda 11 build from source)
- onnx runtime v1.2.0 ~ v1.4.0 对应 cuda 10.1
- onnx runtime v0.5.0 ~ v1.1.2 以下对应 cuda 10.0
- onnx runtime v0.2.1 ~ v0.4.0 以下对应 cuda 9.1 (nuget  packages对应 cuda 10.0)


本机`CUDA`版本是`10.0`，最高可用版本时`v1.1.2`
```sh
pip install onnxruntime-gpu==1.1.2
```