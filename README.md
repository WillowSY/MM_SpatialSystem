# MM_SpatialSystem: Unity Spatial Partitioning

![Unity](https://img.shields.io/badge/Unity-2022.3%2B-black?style=flat&logo=unity)
![C#](https://img.shields.io/badge/Language-C%23-blue?style=flat&logo=csharp)

계층적 해시 그리드를 이용해 Unity C# 환경 메모리 특성에 맞춰 Spatial Partitioning System을 구현했습니다.

---

## 📖 Overview

Unity의 Physics.OverlapSphere 또는 Collider 기반 시스템은 물리 연 비용이 높아 성능 저하의 핵심 원인이 됩니다.
이 코드는 C# 자료구조 기반으로 GC를 최소화하여 O(1)에 가까운 속도로 공간을 조회할 수 있도록 구현한 기능의 주요 부분만 기재한 샘플 코드입니다.

---

## ✨ Key Features

### 1. 자체 공간 분할 시스템 
- AABB/Hash 계산 기반

### 2. O(1)에 가까운 조회 속도
- 월드 좌표 > 해시 Key 변환  
- Dictionary 자료구조 기반으로 바로 조회

### 3. 맵 미리 정의할 필요 없음
- QuadTree/BVH처럼 맵 전체를 미리 정의할 필요 없음  
- **객체가 존재하는 셀만 생성**

### 4. LOD 기반 계층 구조 (Hierarchical Levels)
- 작은 오브젝트용 그리드 : 10m 단위
- 넓은 범위 판정용 그리드 : 100m 단위

---

## 🛠 Optimization
---

### 1) Boxing 방지

**문제점**  
구조체를 Dictionary Key로 사용하면 **박싱 + 힙 할당 유발**.

**해결**  
- `IEquatable<T>` 직접 구현해 박싱 없이 값 비교 가능  
- Prime Number 기반 해시로 충돌 최소화

```csharp
// MM_SpatialHashGrid.cs
private readonly struct CellKey : System.IEquatable<CellKey>
{
    public readonly int X, Y, Z;

    public bool Equals(CellKey other) =>
        X == other.X && Y == other.Y && Z == other.Z;

    public override int GetHashCode() =>
        (X * 73856093) ^ (Y * 19349663) ^ (Z * 83492791);
}
```
### 2) Object Pooling

**문제점**
매 쿼리마다 new List<T>() 하면 GC 누적 발생

**해결**
미리 할당된 버퍼(_queryBuffer)를 재사용하여 런타임 할당 거의 0으로 유지
