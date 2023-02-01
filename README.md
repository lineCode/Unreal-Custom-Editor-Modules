# Unreal Custom Editor Modules

## Module

[Module Properties in Unreal Engine | Unreal Engine 5.1 Documentation](https://docs.unrealengine.com/5.1/en-US/module-properties-in-unreal-engine/)

Unreal 引擎由以许许多多的 Module 构建的，在游戏开发中，可以编写自定义的 Module 来扩展引擎。每个 Module 都封装了一组功能，并且可以提供公共接口和编译环境 (包括宏、路径等) 供其他模块使用。

Module 需要一个以自身模块名为开头的构建文件: `[ModuleName].Build.cs`，在创建工程或插件时会自动生成，默认会将一些必须的模块名称添加到公共依赖模块名称中，其他的按需添加。

```csharp
// Copyright Epic Games, Inc. All Rights Reserved.

using UnrealBuildTool;

public class SuperManager : ModuleRules
{
	public SuperManager(ReadOnlyTargetRules Target) : base(Target)
	{
		PCHUsage = ModuleRules.PCHUsageMode.UseExplicitOrSharedPCHs;

		PublicIncludePaths.AddRange(
			new string[]
			{
				// ... add public include paths required here ...
			}
		);


		PrivateIncludePaths.AddRange(
			new string[]
			{
				System.IO.Path.GetFullPath(Target.RelativeEnginePath) + "Source/Editor/Blutility/Private"
				// ... add other private include paths required here ...
			}
		);


		PublicDependencyModuleNames.AddRange(
			new string[]
			{
				"Core", "Blutility"
				// ... add other public dependencies that you statically link with here ...
			}
		);


		PrivateDependencyModuleNames.AddRange(
			new string[]
			{
				"CoreUObject",
				"Engine",
				"Slate",
				"SlateCore",
				// ... add private dependencies that you statically link with here ...	
			}
		);


		DynamicallyLoadedModuleNames.AddRange(
			new string[]
			{
				// ... add any modules that your module loads dynamically here ...
			}
		);
	}
}
```

插件 Module 可以在 `[MoudleName].uplugin` 中配置其加载时机：

[ELoadingPhase::Type | Unreal Engine Documentation](https://docs.unrealengine.com/5.0/en-US/API/Runtime/Projects/ELoadingPhase__Type/)

```c++
namespace ELoadingPhase
{
    enum Type
    {
        EarliestPossible,
        PostConfigInit,
        PostSplashScreen,
        PreEarlyLoadingScreen,
        PreLoadingScreen,
        PreDefault,				// 在 Game Module 之前
        Default,				// 默认加载点（引擎初始化期间，在 Game Module 之后）
        PostDefault,
        PostEngineInit,
        None,					// 不自动加载
        Max,
    }
}
```

Custom editor functionalities 2 type:

Actions to assets

Actions to actors
