---
layout: timemachine
title: Unity 某个相机截图
tags: unity 
date: 2018-04-01
category: unity
---
# Unity 某个相机截图

```C#
using System.Collections;
using System.Collections.Generic;
using System.IO;
using UnityEngine;

[RequireComponent(typeof(Camera))]
public class ScreenShot : MonoBehaviour {
	
	void OnGUI () {
        if (GUILayout.Button("截图")) {
            var camera = GetComponent<Camera>();
            RenderTexture rt = new RenderTexture(Screen.width, Screen.height, 16);
            camera.targetTexture = rt;
            RenderTexture.active = rt;
            camera.Render();//必须在这里强制渲染一下，否则不会有图像


            Texture2D t = new Texture2D(Screen.width, Screen.height);
            t.ReadPixels(new Rect(0, 0, Screen.width, Screen.height), 0, 0);
            t.Apply();


            var bytes = ImageConversion.EncodeToPNG(t);
            var file = File.Open(Application.dataPath + "/robot.png", FileMode.Create);
            var binary = new BinaryWriter(file);
            binary.Write(bytes);
            file.Close();

            camera.targetTexture = null;
            RenderTexture.active = null;
            Destroy(rt);
        }
	}
}

```
