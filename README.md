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

### Select

自定义 Editor Function 主要分两类：

1. Actions to assets (`UAssetActionUtility : UEditorUtilityObject, IEditorUtilityExtension`)
2. Actions to actors

Scripring Libraries 分为两类：

1. UEditorUtilityLibrary
2. UEditorAssetLibrary

关键方法：

```c++
// Engine\Source\Editor\Blutility\Public\EditorUtilityLibrary.h
// Gets the set of currently selected assets
UFUNCTION(BlueprintCallable, Category = "Development|Editor")
static TArray<UObject*> GetSelectedAssets();

// Gets the set of currently selected classes
UFUNCTION(BlueprintCallable, Category = "Development|Editor")
static TArray<UClass*> GetSelectedBlueprintClasses();

// Gets the set of currently selected asset data
UFUNCTION(BlueprintCallable, Category = "Development|Editor")
static TArray<FAssetData> GetSelectedAssetData();
```

```c++
// Engine\Plugins\Editor\EditorScriptingUtilities\Source\EditorScriptingUtilities\Public\EditorAssetLibrary.h

/**
 * Return the list of all the assets found in the DirectoryPath.
 * @param  DirectoryPath     Directory path of the asset we want the list from.
 * @param  bRecursive       The search will be recursive and will look in sub folders.
 * @param  bIncludeFolder    The result will include folders name.
 * @return The list of asset found.
 */
UFUNCTION(BlueprintCallable, Category = "Editor Scripting | Asset")
static TArray<FString> ListAssets(const FString& DirectoryPath, bool bRecursive = true, bool bIncludeFolder = false);
```

## Automatically Asset Prefixes

[Recommended Asset Naming Conventions in Unreal Engine projects | Unreal Engine 5.1 Documentation](https://docs.unrealengine.com/5.1/en-US/recommended-asset-naming-conventions-in-unreal-engine-projects/)

关键方法：

```c++
// Engine\Source\Editor\Blutility\Public\EditorUtilityLibrary.h
// Renames an asset (cannot move folders)
UFUNCTION(BlueprintCallable, Category = "Development|Editor")
static void RenameAsset(UObject* Asset, const FString& NewName);
```

类型映射表：

```c++
TMap<UClass*, FString> PrefixMap =
{
	{UBlueprint					::StaticClass(), TEXT("BP_")},
	{UStaticMesh				::StaticClass(), TEXT("SM_")},
	{UMaterial					::StaticClass(), TEXT("M_")},
	{UMaterialInstanceConstant	::StaticClass(), TEXT("MI_")},
	{UMaterialFunctionInterface	::StaticClass(), TEXT("MF_")},
	{UParticleSystem			::StaticClass(), TEXT("PS_")},
	{USoundCue					::StaticClass(), TEXT("SC_")},
	{USoundWave					::StaticClass(), TEXT("SW_")},
	{UTexture					::StaticClass(), TEXT("T_")},
	{UTexture2D					::StaticClass(), TEXT("T_")},
	{UUserWidget				::StaticClass(), TEXT("WBP_")},
	{USkeletalMeshComponent		::StaticClass(), TEXT("SK_")},
	{UNiagaraSystem				::StaticClass(), TEXT("NS_")},
	{UNiagaraEmitter			::StaticClass(), TEXT("NE_")}
};
```
