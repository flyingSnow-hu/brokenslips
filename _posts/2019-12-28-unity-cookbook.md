---
layout: article
title: Unity 锦囊
tags: unity cookbook
date: 2019-12-28
category: unity
---
# Shader

## 获取屏幕坐标
```c
o.screenPos = ComputeScreenPos(o.vertex);
```

注意这个坐标：  
  z 无效，一般可以用 COMPUTE_EYEDEPTH 来存深度  
  是投影之前的坐标值，没有除以 w 分量，在 ps 中使用时需要除以w分量才是真正的屏幕坐标  
{:.success}

## 获取深度
### 获取当前像素的深度：   
`define COMPUTE_EYEDEPTH(o) o = -UnityObjectToViewPos( v.vertex ).z`

### 获取深度图里的深度:
1. 在 C# 侧设置相机  
`Camera.main.depthTextureMode = DepthTextureMode.Depth;`

2. 在 Shader 里声明变量  
`sampler2D _CameraDepthTexture;`

3. 注意所使用的 uv  
`LinearEyeDepth(tex2Dproj(_CameraDepthTexture,UNITY_PROJ_COORD(i.screenPos)).r);`

## Shader 支持阴影

### 投影物体
加 shadowcaster 的 Pass，或者直接 _Fallback "VertexLit"_

```c
	Pass {
		Tags { "LightMode"="ShadowCaster" }
		CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#pragma multi_compile_shadowcaster
			#include "UnityCG.cginc"
			sampler2D _Shadow;

			struct v2f{
				V2F_SHADOW_CASTER;
				// float2 uv:TEXCOORD2;
			};

			v2f vert(appdata_base v){
				v2f o;
				// o.uv = v.texcoord.xy;
				TRANSFER_SHADOW_CASTER_NORMALOFFSET(o);
				return o;
			}

			float4 frag( v2f i ) : SV_Target
			{
				// fixed alpha = tex2D(_Shadow, i.uv).a;
				// clip(alpha - 0.5);
				SHADOW_CASTER_FRAGMENT(i)
			}
		ENDCG
	}
```

### 接受阴影的物体：
```c
#include "AutoLight.cginc"
...
stuct v2f {
	LIGHTING_COORDS(0,1) // replace 0 and 1 with the next available TEXCOORDs in your shader, don't put a semicolon at the end of this line.
} 
...
void vert(){	
	// in vert shader;
	TRANSFER_VERTEX_TO_FRAGMENT(o); // Calculates shadow and light attenuation and passes it to the frag shader.
}
...
void frag(){	
	UNITY_LIGHT_ATTENUATION(attenuation,i,i.worldPosition.xyz); 
}
```

### TBN
```c
o.worldPosition = mul(unity_ObjectToWorld,v.vertex);
o.worldNormal = UnityObjectToWorldNormal(v.normal);
o.tangent = UnityObjectToWorldDir(v.tangent.xyz);
o.binormal = cross(o.worldNormal.xyz,o.tangent.xyz)* v.tangent.w * unity_WorldTransformParams.w;

...

half3 normal = TangentNormal(xz);
normal = normalize(
     normal.x * v.tangent +
     normal.y * v.binormal +
     normal.z * v.worldNormal
);
```

## 获取相机透视参数
_ZBufferParams  
```c
float4 _ZBufferParams;// (0-1 range)
// x is 1.0 - (camera's far plane) / (camera's near plane)
// y is (camera's far plane) / (camera's near plane)
// z is x / (camera's far plane)
// w is y / (camera's far plane)
```

## 纹理太多sampler超标了怎么办

### 1. 合并 sampler
```c
#pragma target 4.0 // 前提：4.0 以上

// 替换 sampler2d _Tex;
UNITY_DECLARE_TEX2D(_Tex); (占用 sampler 的图)
UNITY_DECLARE_TEX2D_NOSAMPLER(_NoSampleTex); (不占用 sampler 的图)

// 替换 tex2d(_Tex,i.uv);
UNITY_SAMPLE_TEX2D(_Tex, i.uv);(占用 sampler 的图)
UNITY_SAMPLE_TEX2D_SAMPLER(_NoSampleTex,_Tex, i.uv);(不占用 sampler 的图)
```

### 2. 使用 Texture2DArray
```c
_Tex ("Texture", 2DArray) = "white" {}

#pragma target 3.0
#pragma require 2darray

UNITY_DECLARE_TEX2DARRAY(_Tex);

_Tex.Sample(sampler_Tex,float3(i.uv, texID));
```

## whiteout 法线混合
[解释看这里](http://www.jackcaron.com/techart/2014/11/14/ue4-normal-blending)

`n = UnpackNormal(normalize(a.x + b.x, a.y + b.y, a.z * b.z))`

## 神秘海域 ToneMapping
```c
    float3 ACESFilm( float3 x )
    {
        float a = 2.51f;
        float b = 0.03f;
        float c = 2.43f;
        float d = 0.59f;
        float e = 0.14f;
        return saturate((x*(a*x+b))/(x*(c*x+d)+e));

    }
```

## 用 RGBA 编码 float
```c
inline float4 EncodeFloatRGBA( float v ) {
	float4 enc = float4(1.0, 255.0, 65025.0, 16581375.0) * v;
	enc = frac(enc);
	enc -= enc.yzww * float4(1.0/255.0,1.0/255.0,1.0/255.0,0.0); 
	return enc;
} 

inline float DecodeFloatRGBA( float4 rgba ) { 
	return dot( rgba, float4(1.0, 1/255.0, 1/65025.0, 1/16581375.0) );
}
```

## gpu instancing shader 格式速查
（待补）

## 求亮度

```
float rgb2luma(vec3 rgb) {
    return rgb.g; // Nvidia官方版本
    // return dot(rgb, vec3(0.2126, 0.7152, 0.0722)); // 最流行的亮度计算
    // return dot(rgb, vec3(0.299, 0.587, 0.114)); // 曾经最流行的方法
    // return sqrt(0.299 * rgb.r * rgb.r + 0.587 * rgb.g * rgb.g + 0.114 * rgb.b * rgb.b); // 更精确的计算
    // return sqrt(dot(rgb, vec3(0.299, 0.587, 0.114))); // 添加了 gamma 校正的计算
}
```
UnityCG.cginc 中有 LinearRgbToLuminance 方法，用的是上面第二种

# Editor

## 鼠标点击获取屏幕坐标

`Input.mousePosition`

## 修改鼠标样式
在编辑器内：
```
private void OnSceneGUI(){
    EditorGUIUtility.AddCursorRect (...)
}
```

运行状态：
`Cursor.SetCursor()`

## CustomEditor 多选物体

```c#
[CustomEditor(typeof(MyPlayer))]
[CanEditMultipleObjects]

// 这时只能用 SerializedProperty
OnInspectorGUI(){
        serializedObject.Update();
....
        serializedObject.ApplyModifiedProperties();
    }
```

## 编辑器编译结束事件
```c#
[UnityEditor.Callbacks.DidReloadScripts]
```

## 监听Unity 资源变动
```c#
public class CAssetPostprocessor : AssetPostprocessor
```

## 烘焙相关隐藏变量

可以参考代码通过序列化的方式修改属性，这些属性没有暴露出来。

```c#
 UnityEditor.SerializedObject Sob = new UnityEditor.SerializedObject(r);
 UnityEditor.SerializedProperty Sprop = Sob.FindProperty("m_LightmapParameters");
 Sprop.objectReferenceValue = yourLightmapParameters;
 Sob.ApplyModifiedProperties();
```

Lightmap Parameters 的属性名为 m_LightmapParameters  
Optimize Realtime UVs 的属性名为 m_PreserveUVs，注意值为 false 表示 Optimize Realtime UVs 开启  
Max Distance 的属性名为 m_AutoUVMaxDistance  
Max Angle 的属性名为 m_AutoUVMaxAngle Ignore Normals 的属性名为 m_IgnoreNormalsForChartDetection  
Min Chart Size 的属性名为 m_MinimumChartSize，类型为 int，值为 0 或 1  
scale in lightmap 的属性名为 m_ScaleInLightmap  
{:.success}

## 如何连接 Android Profiler
[知乎](https://zhuanlan.zhihu.com/p/30247546)

 adb forward tcp:55000 localabstract:Unity-com.pwrd.terraindemo 
{:.success}

## 编辑器中用程序启动播放模式
`EditorApplication.ExecuteMenuItem("Edit/Play"); //(2018)`
`EditorApplication.EnterPlayMode() (2019)`

## 编辑器下修改代码之后触发事件
`UnityEditor.Callbacks.DidReloadScripts`

## 编译dll
`UnityEditor.Compilation.AssemblyBuilder`

## 代码驱动清理 Console

```c#
private void ClearConsole () {
	// This simply does "LogEntries.Clear()" the long way:
	var logEntries = System.Type.GetType("UnityEditor.LogEntries,UnityEditor.dll");
	var clearMethod = logEntries.GetMethod("Clear", System.Reflection.BindingFlags.Static | System.Reflection.BindingFlags.Public);
	clearMethod.Invoke(null,null);

	Debug.Log("清理Console");
}
```

## 去掉Mat 上无效的引用
```c#
      public static void CleanUnusedTextures(string materialPath)
      {
      	var mat = AssetDatabase.LoadAssetAtPath<Material>(materialPath);
        var so = new SerializedObject(mat);
        var shader = mat.shader;
        var activeTextureNames = Enumerable
        	.Range(0, ShaderUtil.GetPropertyCount(shader))
            .Where(
                index =>
                ShaderUtil.GetPropertyType(shader, index)
                == ShaderUtil.ShaderPropertyType.TexEnv)
            .Select(index => ShaderUtil.GetPropertyName(shader, index));
        var activeTextureNameSet = new HashSet<string>(activeTextureNames);
        var texEnvsSp = so.FindProperty("m_SavedProperties.m_TexEnvs");
        for (var i = texEnvsSp.arraySize - 1; i >= 0; i--)
        {
            var texSp = texEnvsSp.GetArrayElementAtIndex(i);
            var texName = texSp.FindPropertyRelative("first").stringValue;
            if (!string.IsNullOrEmpty(texName))
            {
                if (!activeTextureNameSet.Contains(texName))
                {
                    texEnvsSp.DeleteArrayElementAtIndex(i);
                }
            }
        }
        so.ApplyModifiedProperties();
    }
```

## 怎么注册回调函数在BUILD PLAYER之前触发

继承 IPreProcessBuildWithReport 并且重写函数 OnPreProcessBuild()

## 在 Project/Hierarchy 面板的每个条目上显示额外信息

EditorApplication.projectWindowItemOnGUI  
EditorApplication.projectWindowItemOnGUI  

本来显示的名字不能修改，可以在同一行右边显示一些信息，传入的 selectionRect 参数表示此行最右边宽度为 0 的一小条区域

## Texture2D 导出文件

```c#
	Texture2D t;
	// ...
	var bytes = ImageConversion.EncodeToPNG(t);
	var file = File.Open(Application.dataPath + "/robot.png", FileMode.Create);
	var binary = new BinaryWriter(file);
	binary.Write(bytes);
	file.Close();
```

## 转换 ASTC 格式
（待补）

# 常用算法

## 经纬度和欧拉角转换
```c#
   private Vector2 DirToLLCoord(Vector3 dir){
       var longitude /* 经度，以正前方为0 */ = Mathf.Atan2(dir.x,dir.z);
       var latitude /* 纬度 */ = Mathf.Acos(dir.y);
       return new Vector2(longitude,latitude);
   }

   private Vector3 LLCoordToDir(Vector2 llCoord){
       return new Vector3(
           Mathf.Sin(llCoord.y) * Mathf.Sin(llCoord.x),
           Mathf.Cos(llCoord.y),
           Mathf.Sin(llCoord.y) * Mathf.Cos(llCoord.x)
           );
   }
```

## 平面上一点是否在三角形内
```c#
bool PointinTriangle(Vector3 A, Vector3 B, Vector3 C, Vector3 P)
{
	Vector3 v0 = C - A ;
	Vector3 v1 = B - A ;
	Vector3 v2 = P - A ;

	float dot00 = v0.Dot(v0) ;
	float dot01 = v0.Dot(v1) ;
	float dot02 = v0.Dot(v2) ;
	float dot11 = v1.Dot(v1) ;
	float dot12 = v1.Dot(v2) ;

	float inverDeno = 1 / (dot00 * dot11 - dot01 * dot01) ;

	float u = (dot11 * dot02 - dot01 * dot12) * inverDeno ;
	if (u < 0 || u > 1) // if u out of range, return directly
	{
		return false ;
	}

	float v = (dot00 * dot12 - dot01 * dot02) * inverDeno ;
	if (v < 0 || v > 1) // if v out of range, return directly
	{
		return false ;
	}

	return u + v <= 1 ;
}
```

## mesh 导出 obj 文件
```c#
public string MeshToString(MeshFilter mf) {
	Mesh m = mf.sharedMesh;
	Material[] mats = mf.GetComponent<Renderer>().sharedMaterials;
	StringBuilder sb = new StringBuilder();
	sb.Append("g ").Append(mf.name).Append("\n");

	foreach(Vector3 v in m.vertices) {
		//x取负是为了调换坐标系手性，乘100是用于导入max
	    sb.Append(string.Format("v {0} {1} {2}\n",100*(-v.x+Center.x),100*(v.y-Center.y),100*(v.z-Center.z)));
	}
	sb.Append("\n");

	foreach(Vector3 v in m.normals) {
		sb.Append(string.Format("vn {0} {1} {2}\n",-v.x,v.y,v.z));
	}
	sb.Append("\n");

	foreach(Vector3 v in m.uv) {
		sb.Append(string.Format("vt {0} {1}\n",v.x,v.y));
	}

	for (int material=0; material < m.subMeshCount; material ++) {
		sb.Append("\n");
		sb.Append("usemtl ").Append(mats[material].name).Append(".mat\n");
		sb.Append("usemap ").Append(mats[material].name).Append(".mat\n");
		int[] triangles = m.GetTriangles(material);
		for (int i=0;i<triangles.Length;i+=3) {
			sb.Append(string.Format("f {0}/{0}/{0} {1}/{1}/{1} {2}/{2}/{2}\n",
			triangles[i]+1, triangles[i+2]+1, triangles[i+1]+1));
		}
	}
	return sb.ToString();
}
public void MeshToFile(MeshFilter mf, string filename) {
	using (StreamWriter sw = new StreamWriter(filename))
	{
	sw.Write(MeshToString(mf));
	}
}
```

## 如何读写Excel
```c#
IWorkbook workbook = new HSSFWorkbook ();
ISheet sheet = workbook.CreateSheet ();
```

## 为什么 Surface Shader 里要  atten * 2

[Aras 大神的解释](https://forum.unity.com/threads/why-atten-2.94711/#post-1134403)

Short answer to "why multiply by two?" - because in the EarlyDays, it was a cheap way to "fake" somewhat overbright light in fixed function shaders. And then it stuck, and it kind of dragged along.
We'd like to kill this concept. But that would mean breaking possibly a lot of existing shaders that all of you wrote (suddenly everything would become twice as bright). So yeah, a tough one... worth killing an old stupid concept or not?
{:.success}

# 技巧

## ⚠无法删掉的 MonoBehaviour
像这样写一段代码，就能让 MonoBehaviour 一旦被添加到物体上，就永远无法删掉（亲测 2019.4.18.f1 有效）
```c#
	using UnityEngine;
	
	[RequireComponent(typeof(CannotRemove))]
	public class CannotRemove:MonoBehaviour
	{
		// ...
	}
```
无论是通过编辑器还是通过脚本，都无法删掉这个组件——当然你还是可以 Destroy 整个物体  
如果想更绝情一点，还可以加一个 DisallowMultipleComponent
  
  
## 地形 Terrain 不能旋转和缩放

 所以地形 Shader 里获取的 normal 不需要 UnityObjectToWorldNormal，因为只是平移不会修改法线，Object 即 World  
 切线可以固定为 (1,0,0)，简化 tbn 矩阵转换

```c
    // 简化： half3 tangent = half3(1,0,0);
    normal = half3(
        tanNormal.x + worldNormal.x * tanNormal.z,
        worldNormal.y * tanNormal.z - worldNormal.z * tanNormal.y,
        dot(worldNormal.yz,tanNormal.yz)
    );
```
