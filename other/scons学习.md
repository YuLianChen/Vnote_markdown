# scons学习
1. 如果编译出错，依赖没有实现，会直接返回空列表
2. 检查子目录是否正确编译
```python
print('=====================================================current dir:', cwd )
lv_demos_src = SConscript(os.path.join(cwd, 'lv_demos/SConscript'), variant_dir="lv_demos", duplicate=0)
print("lv_demos src:", lv_demos_src)  # 检查子目录是否返回了文件
src.extend(lv_demos_src)
```
3. 