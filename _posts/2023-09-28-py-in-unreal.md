---
layout: article
title: Python In Unreal
tags: unreal ue python
date: 2023-09-28
category: unreal
---
# Unreal 锦囊

## 少见的接口

```C++
    // 排序
    // 假如有一个列表
    TArray<FItem> ItemList;
    Algo::SortBy(EntryList, &FItem::ID);
    Algo::SortBy(EntryList, &FItem::SomeField, TGreater<>());
```

## Python

**问题：** 如何在 ue 使用 asyncio
**解决：** 正常地使用 asyncio loop 会卡住 ue 主线程，需要使用ue本身的tick

```python
def update(delta_seconds):
    try:
        loop = asyncio.get_event_loop()
        if not loop.is_running():
            loop.call_soon(loop.stop)
            loop.run_forever()
    except Exception as e:
        unreal.unregister_slate_post_tick_callback(self.handle)
        raise e

handle = unreal.register_slate_post_tick_callback(update)
```


**问题：** 如何在 ue 启用 PyQt
**解决：** 同样，正常地使用 pyqt 会卡住 ue 主线程，需要使用ue本身的tick

```python
    def main():
        if not QApplication.instance():
            app = QApplication(sys.argv)
        else:
            app = QApplication.instance()

        window = MainWindow()
        window.show()

        return window

    window = MainWindow.main()
    window.setObjectName('EditorWindow')
    unreal.parent_external_window_to_slate(window.winId())
```

