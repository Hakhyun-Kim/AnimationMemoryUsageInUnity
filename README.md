# Animation Memory Usage in Unity [![License](https://img.shields.io/badge/License-MIT-lightgrey.svg?style=flat)](http://mit-license.org)
애니메이션 메모리 사용량 최적화 사례 - 이 리포지토리는 다양한 유형의 3D 애니메이션을 Unity로 가져올 때 최적의 메모리 사용을 위해서 모든 가능한 결과를 분석하려고 합니다. 애니메이션은 일반적으로 프로젝트에서 가장 메모리를 많이 사용하는 Asset이 아니지만 많은 3D 모바일 게임은 애니메이션이 매우 많기 때문에 최적화가 필요할 수 있습니다.

## Conclusions
이 섹션은 데이터를 통하여 확인된 모든 내용을 포함합니다.

![](/Screenshots/WalkAnimation.gif)

### Summary
복잡한 Rig(DCC 패키지에서 애니메이션을 구현하려면 복잡하게 됨)를 사용하여 가능한 최소한의 키를 사용하여 간단한 단일 계층 Rig Skeleton 및 애니메이션을 생성하고 _Animation Compression_을 다음으로 설정하여 _Unity_에서 가져오는 것이 가장 좋습니다. **최적**. _애니메이션 마스킹_ 기능을 사용하여 빌드에서 애니메이션의 크기를 더 줄일 수 있습니다. 

동일한 애니메이션에 대해 이 테스트에서 이용 된 가장 최적화된 상황과 그 반대 상황 간의 차이는 500KB 이상이었습니다. 각 캐릭터당 20개의 애니메이션이 있고 동시에 로드되는 10개의 캐릭터가 있는 경우 거의 100MB의 런타임 메모리 낭비가 발생할 수 있습니다. 3D 프로젝트의 경우 텍스처가 일반적으로 메모리 사용에 훨씬 더 문제가 되는 경우가 많습니다만 애니메이션도 갯수가 많은 경우 제한을 초과할 수 있기 때문에 반드시 확인해야 합니다.

### Detailed Explanation 
애니메이션 클립의 메모리 사용량을 최적화 하려면 일반적으로 다음을 권장합니다.
1. [Unity Avatar](https://docs.unity3d.com/Manual/class-Avatar.html)를 생성/정의하는 데 사용되는 Skeleton/Rig는 Rig와 Mesh를 포함해야 하지만 애니메이션은 포함하지 않아야 합니다. 아래 이미지는 왼쪽에 계층 구조가 있고 "단순화"/단일 계층 구조(오른쪽에 Unity로 내보낼 준비가 됨)가 있는 전체 Rig를 보여줍니다.

![](/Screenshots/RigInDCC.png)

더 나아가 위 이미지의 오른쪽은 모델과 단일 계층 체인 Rig(_Unity_에서 볼 수 있음)만 포함하는 Skeleton .fbx 파일을 아래 이미지에서 확인할 수 있습니다.
![](/Screenshots/SkeletonContents.png)

2. 리그에서 생성된 _Avatar_는 해당 Rig를 공유해야 하는 모든 애니메이션에 사용해야 합니다. 그렇지 않으면 각 애니메이션이 자체 아바타로 끝나고 추가 메모리 사용량이 필요합니다. 애니메이션에 기존 _Avatar_를 사용하려면 사용자가 아래 이미지와 같이 애니메이션에 대한 가져오기 옵션에서 정의해야 합니다. Rig/Character 설정에 대해 _Avatar_를 하나만 만드는 것이 매우 중요합니다. _Avatar_를 만드는 데 사용된 원래 skeleton + Mesh는 해당 Rig에 필요한 모든 모션 또는 Deformation을 달성하는 데 필요한 모든 것을 포함해야 합니다. 리그와 함께 작동하도록 의도된 모든 애니메이션은 기존 _Avatar_를 가리키기만 하면 됩니다. 이 설정을 수행하지 않으면 사용자는 새로운 _Avatar_에 대한 추가 비용이 발생하지만 애니메이션 파일에서 제거되지 않는 외부 데이터(예: 존재하지 않거나 무시되어서는 안 되는 Mesh 또는 Rig 컨트롤러와 같은 외부 데이터)에 대한 추가 비용이 발생할 수 있습니다.

![](/Screenshots/ReuseAvatar.png)

3. 단일 애니메이션 파일에는 메시가 포함되어서는 안 됩니다. 각 애니메이션에는 Rig 조인트와 그에 대한 애니메이션만 포함되어야 합니다(엄밀히 말하면 애니메이션 소스 파일이 아니라 Export 파일에 관한 것임). 리포지토리의 예에서 보여주듯이 애니메이션 파일에 메시를 포함한다고 해서 빌드된 프로젝트의 메모리에 있는 애니메이션 크기가 증가하지 않습니다(애니메이션 데이터만 _Unity 애니메이션 클립_을 생성하는 데 사용되기 때문). 그러나 애니메이션이 상당히 증가합니다. 디스크의 파일 크기. 이 예에서 크기 차이는 파일당 약 300KB입니다. 100 개의 캐릭터를 사용하는 프로젝트에서 캐릭터당 50개의 애니메이션을 곱하면 1.4GB가 넘는 공간이 낭비됩니다.또한 애니메이션 파일에 메시가 있으면 게임내 Scene 에서 해당 메시를 실수로 사용할 위험이 추가됩니다. 이 경우 _Unity_는 메모리에 해당 메시의 복제본을 갖게 되고 실제 런타임 메모리도 300KB 배 증가합니다. 아래 이미지는 애니메이션이 있는 단일 계층 체인 Rig만 포함하는 애니메이션 .fbx 파일의 예입니다. 애니메이션은 이미 생성된 _Avatar_에 적용해야 하기 때문에 Scene에서 직접 드래그하면 장면 미리보기 창에 애니메이션이 표시되지 않는다는 점에 유의하는 것이 중요합니다.

테스트 데이터에서 우리는 두 개의 프로젝트를 만들었습니다. 하나는 메시가 없고 애니메이션이 있는 프로젝트이고 다른 하나는 동일한 메시가 있고 애니메이션이 있는 프로젝트입니다. 결과는 다음과 같습니다.
- 두 프로젝트의 빌드 크기는 27609KB였습니다.
- 메시가 없는 애니메이션은 디스크에서 750KB인 반면 메시가 있는 애니메이션은 디스크에서 1033KB였습니다.
- 둘 다에 대한 애니메이션 클립(1)은 런타임 메모리에서 17KB였습니다.
- 둘 다에 대한 메시(1)는 런타임 메모리에서 372.3KB였습니다. 

결론:
- 애니메이션 파일에 메시가 존재하더라도 이미 존재하는 _Avatar_를 사용하는 한(새로 생성되는 것과 반대), 메시는 메모리에서 복제되지 않고 무시됩니다.
- 애니메이션에 메시가 있으면 런타임 메모리에 영향을 미치지 않을 수 있지만 디스크 크기(프로젝트 크기), 가져오기 시간 및 일반 Editor 성능이 증가할 수 있습니다.

![](/Screenshots/AnimationContents.png)

보편적으로 적용할 수 있는 것은 아니지만 다음 설정을 권장합니다(프로젝트 별로 프로파일링하는 것이 가장 정확합니다).

1.메모리 최소화가 목적이라면 _Anim. 압축_은 **최적**으로 설정해야 합니다. 모바일 게임 및 애플리케이션의 경우 **최적**은 게임 플레이에 사용되는 모든 애니메이션에 가장 적합한 선택입니다(필요한 경우 영화 애니메이션은 덜 압축될 수 있음). 세 가지 애니메이션 압축 옵션은 "Off", "Optimal" 및 "Keyframe Reduction"입니다. 대부분의 경우 "최적"은 최상의 애니메이션 압축에 가장 적합한 옵션으로 메모리 공간을 더 적게 차지합니다. **최적**은 키프레임 축소를 시도하지만 전체 프레임 사이가 더 나빠지기 때문에 애니메이션 품질이 저하됩니다(키 사이의 보간이 나빠지는 대신 메모리가 적음). 각 애니메이션에 대한 각 설정을 미세 조정하면 더 나은 결과를 얻을 수 있지만 수백 개의 애니메이션에 대해 확장할 수 없는 솔루션일 가능성이 높습니다. 이 데모에서는 "최적" 설정을 사용하여 메모리를 크게 줄였습니다. 다음은 몇 가지 예입니다.
- 키가 있는 포즈만 사용하고 "최적" 압축을 사용하여 가져온 짧은 걷기 애니메이션은 "키프레임 축소" 옵션을 사용하여 17KB <>였으며 동일한 애니메이션은 "끄기" 옵션(압축 없음)을 사용하여 43.6KB <>였습니다. 애니메이션은 70KB였습니다.
- "최적" 압축을 사용하여 굽고 가져온 모든 키를 사용하는 긴 걷기 애니메이션은 "키프레임 감소" 옵션을 사용하여 103KB <>였고, 동일한 애니메이션이 "끄기" 옵션(압축 없음)을 사용하여 293.6KB <>였습니다. 애니메이션은 614KB였습니다.
위의 예에서 볼 수 있듯이, 그 차이는 긴(300프레임) 애니메이션에서 500KB가 될 수 있으며, 이는 빠르게 합산될 수 있습니다.

![](/Screenshots/AnimationCompressionOptimal.png)

또한 DCC 패키지에서 가져온 애니메이션 데이터로 작업할 때 몇 가지 일반적인 실수가 있습니다.
1. 각 Rig 및 애니메이션 .fbx 파일에는 일반적으로 단일 계층 조인트 체인이라고 하는 내용만 포함되어야 합니다. 일반적으로 애니메이션 Rig는 DCC 패키지에서 매우 복잡하여 애니메이터가 "기본" 골격을 광범위하게 제어할 수 있습니다. 이 복잡한 설정에는 일반적으로 IK(역운동학) 컨트롤러 및 설정, FK(순운동학) 컨트롤러 및 설정, 도우미 노드 등이 포함됩니다. 이러한 노드 중 일부는 .fbx 형식에서 지원하는 노드이며 _실수로_ 내보내면 빈 노드가 생성됩니다. _GameObjects_는 _Unity_ 계층에 있지만 여전히 애니메이션 데이터를 포함하고 메모리 리소스를 낭비합니다. Skeleton/Rig(아바타 생성에 사용) 모두에 대해 단일 계층 조인트 체인을 **만** 내보내는 것이 좋으며, 애니메이션의 경우 메시에 스키닝되고 모션에서 직접 사용되는 조인트만 내보내는 것이 좋습니다(Helper Joint 없음) - 이 프로세스에는 일반적으로 Rig 전반에 걸친 표준화(이름 지정 또는 예상 콘텐츠/노드)와 게임 엔진에 필요한 것과 필요하지 않은 것을 구별할 수 있는 자동화 솔루션이 필요합니다. 애니메이션을 _Humanoid_로 임포트할 때 _Humanoid_ Bone 매핑이 정크 데이터를 버리기 때문에 시스템에 의해 관련 없는 데이터가 정리될 수 있지만 동일한 정크 데이터가 _Generic_ 가져오기에서 유지될 수 있다는 점에 유의하는 것이 중요합니다. 필요한 데이터만 사용하여 메모리 추가 사용의 위험을 제거하는 것이 좋습니다. 다음은 이 주장을 뒷받침하는 이 분석의 일부 데이터입니다.
- 단일 계층 리그로만 내보내고 _Humanoid_로 가져올 때 스켈레톤은 전체 리그와 함께 내보내고 _Humanoid_로 가져올 때 스켈레톤이 46.3KB였습니다.
- 단일 계층 구조만으로 내보내고 최적의 압축으로 가져올 때 모든 키가 구운 짧은 걷기 애니메이션은 전체 리그로만 내보내고 최적의 압축으로 가져올 때 17KB <>였고 모든 키가 구운 짧은 걷기 애니메이션은 다음과 같았습니다. 70KB
2. 애니메이션은 내보낼 때 모든 키프레임을 굽거나 원본 소스 애니메이션 파일에서 미리 굽도록 설정해서는 안 됩니다. 애니메이션이 원본 소스 파일의 모든 키프레임에 미리 구워지면 파일을 업데이트하고 반복하기가 매우 어렵습니다. 또한 내보낸 결과도 자동으로 구워집니다. 마찬가지로 애니메이션이 내보낼 때 구워지도록 설정되어 있으면 범위의 모든 키프레임에 애니메이션 키가 있어 큰 애니메이션 데이터 세트가 생성됩니다. Unity에는 애니메이션 클립에 있는 데이터를 압축하는 방법(예: [Animation Compression](https://docs.unity3d.com/Manual/class-AnimationClip.html) 가져오기 설정)이 있고 훨씬 더 좋습니다. 필요한 키프레임만 유지되는 경우. _Unity_가 DCC 패키지와 다르게 애니메이션 곡선을 해석하는 방식으로 인해 예술적 품질이 충족되지 않는 경우 사용자는 전체 애니메이션을 베이킹하는 것과 반대로 애니메이션의 문제가 있는 섹션을 선택적으로 베이킹하거나 [_Resample Curves_]( https://docs.unity3d.com/Manual/class-AnimationClip.html) 옵션(옵션은 기본적으로 활성화되어 있습니다. 비활성화하면 "애니메이션 곡선을 원래 작성된 대로 유지"합니다.

애니메이션을 가비지 데이터(예: 애니메이션에 영향을 미치지 않는 컨트롤러 또는 helper 노드)와 함께 _Unity_로 가져오면 _Editor_ workflow 에서 성능 저하가 발생합니다. 프로젝트에서 모든 키가 베이크된 전체 Rig, 긴 애니메이션 걷기 주기(300프레임)에 대한 최악의 시나리오는 디스크에서 21453KB(약 21MB)였습니다. 단순화된 단일 계층 구조 리그에 대한 최상의 시나리오에서 동일한 긴 애니메이션 걷기 주기에 대해 필요한 주요 포즈(및 몇 가지 중간)만 베이크된 동일한 애니메이션은 1865KB(약 1.8MB)였습니다. 최상의 시나리오와 최악의 시나리오의 차이는 애니메이션 파일당 약 19MB입니다. 수백 개의 애니메이션으로 디스크 추가 사용은수 GB에 이릅니다. 또한 더 큰 애니메이션 파일은 가져오기 시간에 영향을 미치고 일반적인 _Editor_ 성능에 영향을 줄 수 있습니다.

마지막으로, Rig와 애니메이션을 가비지 데이터와 함께 Unity로 가져오고 사용자가 소스 파일을 개선할 수 없거나 소스 파일을 개선할 수 없는 경우에도 Unity는 [애니메이션 마스크](https://docs.unity3d.com)를 제공합니다. /Manual/AnimationMaskOnImportedClips.html) 빌드 프로세스 중에 이 데이터를 제거하는 방법입니다. _애니메이션 마스크_는 skeleton/아바타에 연결되어 동일한 _아바타_를 사용하는 모든 애니메이션을 차단하는 데 사용 및 재사용할 수 있는 Unity serialized asset입니다. 위 이미지에서 볼 수 있듯이 마스킹은 휴머노이드에서 시각적으로 수행하거나 변환 계층을 직접 무시하여 수행할 수 있습니다. 빌드 프로세스 동안 마스크된 애니메이션이 클립에서 제거되어 더 가벼운 애니메이션 클립이 생성되지만(메모리 측면에서) 클립의 모든 애니메이션이 손실된다는 점에 유의하는 것이 중요합니다.

![](/Screenshots/AnimationMask.PNG)

이러한 규칙의 대부분은 애니메이션 클립의 메모리 사용량을 최소화해야 하는 대부분의 프로젝트에 적용됩니다. 이러한 결론에 대한 추가 정보 및 기술적인 설명은 아래 **Raw 데이터** 및 **범례** 섹션을 참조하십시오.

## Special Notes
모든 데이터는 iPhone 6s(2015년 출시)에서 실행되는 개발 빌드에서 런타임 시 캡처되었습니다.

## Raw Data
아래 데이터 테이블은 [Unity Memory Profiler](https://docs.unity3d.com/Manual/ProfilerMemory.html)에서 추출한 메모리 사용량의 Raw 데이터를 포함합니다. 모든 애니메이션은 두 개의 게임 개체에 적용되었으며 단일 **메모리 스냅샷**에 함께 캡처되었습니다. 숫자가 있는 세 개의 열은 순서대로(왼쪽에서 오른쪽으로) 다음을 나타냅니다.
- 휴머노이드로 가져올 때 리그/애니메이션의 메모리 내 크기
- 일반으로 가져올 때 리그/애니메이션의 메모리 내 크기
- 디스크의 .fbx 파일 크기

| Animation Clip Name                                                         | Humanoid In Memory Size (KB) | Generic In Memory Size (KB) | On Disk Size (KB) |
|-----------------------------------------------------------------------------|:----------------------------:|:---------------------------:|:-----------------:|
| PolyBod_ShOnly_Skeleton                                                     |             42.5             |             28.5            |        361        |
| PolyBod_ShOnly_Anim_Walk_Short_KeyedPosesOnly_CompressionOptimal            |              17              |             25.9            |        750        |
| PolyBod_ShOnly_Anim_Walk_Short_KeyedPosesOnly_WithMesh_CompressionOptimal   |              17              |              -              |        1033       |
| PolyBod_ShOnly_Anim_Walk_Short_KeyedPosesOnly_CompressionOff                |              70              |            199.4            |        750        |
| PolyBod_ShOnly_Anim_Walk_Short_KeyedPosesOnly_CompressionKeyframeReduction  |             43.6             |             42.5            |        750        |
|                                                                             |                              |                             |                   |
| PolyBod_ShOnly_Anim_Walk_Short_Baked_CompressionOptimal                     |              17              |             26.8            |        1274       |
| PolyBod_ShOnly_Anim_Walk_Short_Baked_CompressionOff                         |              70              |            190.7            |        1274       |
| PolyBod_ShOnly_Anim_Walk_Short_Baked_CompressionKeyframeReduction           |             43.6             |             49.5            |        1274       |
|                                                                             |                              |                             |                   |
| PolyBod_ShOnly_Anim_Walk_Long_KeyedPosesOnly_Optimal                        |             104.7            |            141.4            |        1865       |
| PolyBod_ShOnly_Anim_Walk_Long_KeyedPosesOnly_CompressionOff                 |              614             |             1843            |        1865       |
| PolyBod_ShOnly_Anim_Walk_Long_KeyedPosesOnly_CompressionKeyframeReduction   |             293.6            |            208.6            |        1865       |
|                                                                             |                              |                             |                   |
| PolyBod_ShOnly_Anim_Walk_Long_Baked_CompressionOptimal                      |              103             |            126.4            |       10193       |
| PolyBod_ShOnly_Anim_Walk_Long_Baked_CompressionOff                          |              614             |             1740            |       10193       |
| PolyBod_ShOnly_Anim_Walk_Long_Baked_CompressionKeyframeReduction            |             293.6            |            265.3            |       10193       |
|                                                                             |                              |                             |                   |
| PolyBod_FullRig_Skeleton                                                    |             46.3             |             63.2            |        448        |
| PolyBod_FullRig_Anim_Walk_Short_KeyedPosesOnly_CompressionOptimal           |              17              |             65.7            |        1493       |
| PolyBod_FullRig_Anim_Walk_Short_KeyedPosesOnly_CompressionOff               |             69.4             |            424.4            |        1493       |
| PolyBod_FullRig_Anim_Walk_Short_KeyedPosesOnly_CompressionKeyframeReduction |             39.1             |            111.3            |        1493       |
|                                                                             |                              |                             |                   |
| PolyBod_FullRig_Anim_Walk_Short_Baked_CompressionOptimal                    |              70              |             65.6            |        3135       |
| PolyBod_FullRig_Anim_Walk_Short_Baked_CompressionOff                        |              70              |            411.8            |        3135       |
| PolyBod_FullRig_Anim_Walk_Short_Baked_CompressionKeyframeReduction          |             41.4             |            143.4            |        3135       |
|                                                                             |                              |                             |                   |
| PolyBod_FullRig_Anim_Walk_Long_KeyedPosesOnly_Optimal                       |             104.7            |            449.9            |        4156       |
| PolyBod_FullRig_Anim_Walk_Long_KeyedPosesOnly_CompressionOff                |              614             |             3993            |        4156       |
| PolyBod_FullRig_Anim_Walk_Long_KeyedPosesOnly_CompressionKeyframeReduction  |             293.6            |             700             |        4156       |
|                                                                             |                              |                             |                   |
| PolyBod_FullRig_Anim_Walk_Long_Baked_CompressionOptimal                     |             104.9            |            472.9            |       21453       |
| PolyBod_FullRig_Anim_Walk_Long_Baked_CompressionOff                         |              614             |             3993            |       21453       |
| PolyBod_FullRig_Anim_Walk_Long_Baked_CompressionKeyframeReduction           |             359.4            |             1400            |       21453       |

Below are some of the screenshots of the raw data, directly from the Unity Memory Profiler.
The image below is a screenshot of the raw data in the Memory Profiler when using a Humanoid import.

![](/Screenshots/DeviceMemoryUsage_Humanoid.PNG)

The image below is a screenshot of the raw data in the Memory Profiler when using a Generic import.

![](/Screenshots/DeviceMemoryUsage_Generic.png)

_It is important to note that the Humanoid animations seem to be better optimized for memory than the Generic animations and that is because Unity, at build time, scraps the data that doesn't map to the Humanoid Avatar and it the process reduces memory usage. The same optimization result can be achieved on the Generic animations by either improving the import data in the DCC package or by using Animation Masking in Unity._

## Legend
The .fbx rig and animation files were generated using Autodesk Maya 2020, but the same measurements and rules apply for rigs and animations generated from any DCC packages. The rig was generated using Maya's HumanIK System (HIK), but the same concepts apply to any rig system.

Below is a quick explanation of what each file contains:
- **PolyBod_ShOnly_Skeleton** = The single hierarchy joints only rig + mesh
- **PolyBod_ShOnly_Anim_Walk_Short_KeyedPosesOnly_CompressionOptimal** = The single hierarchy joints only rig + no mesh + short walk animation with key poses only
- **PolyBod_ShOnly_Anim_Walk_Short_KeyedPosesOnly_WithMesh_CompressionOptimal** = The single hierarchy joints only rig + no mesh + short walk animation with key poses only - imported with Animation Compression set to Optimal
- **PolyBod_ShOnly_Anim_Walk_Short_KeyedPosesOnly_CompressionOff** = The single hierarchy joints only rig + no mesh + short walk animation with key poses only - imported with Animation Compression set to Off
- **PolyBod_ShOnly_Anim_Walk_Short_KeyedPosesOnly_CompressionKeyframeReduction** = The single hierarchy joints only rig + no mesh + short walk animation with key poses only - imported with Animation Compression set to Keyframe Reduction

- **PolyBod_ShOnly_Anim_Walk_Short_Baked_CompressionOptimal** = The single hierarchy joints only rig + no mesh + short walk animation with all keyframes in range baked - imported with Animation Compression set to Optimal
- **PolyBod_ShOnly_Anim_Walk_Short_Baked_CompressionOff**  = The single hierarchy joints only rig + no mesh + short walk animation with all keyframes in range baked - imported with Animation Compression set to Off
- **PolyBod_ShOnly_Anim_Walk_Short_Baked_CompressionKeyframeReduction** = The single hierarchy joints only rig + no mesh + short walk animation with all keyframes in range baked - imported with Animation Compression set to Keyframe Reduction

- **PolyBod_ShOnly_Anim_Walk_Long_KeyedPosesOnly_Optimal** = The single hierarchy joints only rig + no mesh + long walk animation with key poses only  - imported with Animation Compression set to Optimal
- **PolyBod_ShOnly_Anim_Walk_Long_KeyedPosesOnly_CompressionOff** = The single hierarchy joints only rig + no mesh + long walk animation with key poses only - imported with Animation Compression set to Off
- **PolyBod_ShOnly_Anim_Walk_Long_KeyedPosesOnly_CompressionKeyframeReduction** = The single hierarchy joints only rig + no mesh + long walk animation with key poses only - imported with Animation Compression set to Keyframe Reduction

- **PolyBod_ShOnly_Anim_Walk_Long_Baked_CompressionOptimal** = The single hierarchy joints only rig + no mesh + long walk animation with all keyframes in range baked - imported with Animation Compression set to Optimal
- **PolyBod_ShOnly_Anim_Walk_Long_Baked_CompressionOff**  = The single hierarchy joints only rig + no mesh + long walk animation with all keyframes in range baked - imported with Animation Compression set to Off
- **PolyBod_ShOnly_Anim_Walk_Long_Baked_CompressionKeyframeReduction**  = The single hierarchy joints only rig + no mesh + long walk animation with all keyframes in range baked - imported with Animation Compression set to Keyframe Reduction

- **PolyBod_FullRig_Skeleton** = The full hierarchy joints rig (controllers, dummy nodes, single hierarchy rig) + mesh
- **PolyBod_FullRig_Anim_Walk_Short_KeyedPosesOnly_CompressionOptimal** = The full hierarchy rig + no mesh + short walk animation with key poses only - imported with Animation Compression set to Optimal
- **PolyBod_FullRig_Anim_Walk_Short_KeyedPosesOnly_CompressionOff** = The full hierarchy rig + no mesh + short walk animation with key poses only  - imported with Animation Compression set to Off
-**PolyBod_FullRig_Anim_Walk_Short_KeyedPosesOnly_CompressionKeyframeReduction** = The full hierarchy rig + no mesh + short walk animation with key poses only - imported with Animation Compression set to Keyframe Reduction

- **PolyBod_FullRig_Anim_Walk_Short_Baked_CompressionOptimal** = The full hierarchy rig + no mesh + short walk animation with all keyframes in range baked  - imported with Animation Compression set to Optimal
- **PolyBod_FullRig_Anim_Walk_Short_Baked_CompressionOff** = The full hierarchy rig + no mesh + short walk animation with all keyframes in range baked - imported with Animation Compression set to Off
- **PolyBod_FullRig_Anim_Walk_Short_Baked_CompressionKeyframeReduction**  = The full hierarchy rig + no mesh + short walk animation with all keyframes in range baked - imported with Animation Compression set to KeyframeReduction

- **PolyBod_FullRig_Anim_Walk_Long_KeyedPosesOnly_Optimal**  = The full hierarchy rig + no mesh + long walk animation with only key poses  - imported with Animation Compression set to Optimal
- **PolyBod_FullRig_Anim_Walk_Long_KeyedPosesOnly_CompressionOff**  = The full hierarchy rig + no mesh + long walk animation with only key poses  - imported with Animation Compression set to Off
- **PolyBod_FullRig_Anim_Walk_Long_KeyedPosesOnly_CompressionKeyframeReduction**  = The full hierarchy rig + no mesh + long walk animation with only key poses  - imported with Animation Compression set to Keyframe Reduction

- **PolyBod_FullRig_Anim_Walk_Long_Baked_CompressionOptimal** = The full hierarchy rig + no mesh + long walk animation with all keyframes in range baked - imported with Animation Compression set to Optimal
- **PolyBod_FullRig_Anim_Walk_Long_Baked_CompressionOff** = The full hierarchy rig + no mesh + long walk animation with all keyframes in range baked - imported with Animation Compression set to Off
- **PolyBod_FullRig_Anim_Walk_Long_Baked_CompressionKeyframeReduction**  = The full hierarchy rig + no mesh + long walk animation with all keyframes in range baked - imported with Animation Compression set to Keyframe Reduction

All results and data obtained have been collected from the project generated with the data in this repository and can be replicated by profiling a development build generated from the project provided in the repository.
